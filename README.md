# Incremental Summary Analysis - EUF AI Summarizer Service

## Overview
This document provides a comprehensive analysis of how incremental summaries work in the EUF AI Free Text Summarizer service, specifically for AI Drive and DEX campaigns.

---

## What is Incremental Summary?

Incremental summary is a feature that allows the summarizer to build upon previous summaries instead of regenerating summaries from scratch each time. This approach:
- **Reduces AI costs** by processing only new data with context from previous summaries
- **Improves consistency** by maintaining context across multiple summary generations
- **Enables continuous learning** by storing and updating summary states

---

## How Incremental Summary Works

### 1. Architecture Components

#### Key Processor: `IncrementalSummaryActionFixedKeyProcessor`
- **Location**: `engage.euf-ai-free-text-summarizer/src/main/java/com/nexthink/euf/freetext/summarizer/kstream/processor/IncrementalSummaryActionFixedKeyProcessor.java`
- **Purpose**: Handles incremental summary actions by comparing current summaries with stored previous summaries

#### State Store: `SUMMARY_ACTION_INCREMENTAL_STORE`
- **Name**: `summary-action-incremental-store-v2`
- **Type**: KeyValueStore<String, EngageCommentsSummaryActionDTOTrim>
- **Purpose**: Stores trimmed versions of previous summaries for comparison

### 2. Processing Flow

```
┌─────────────────────────────────────────────────────────────┐
│  1. New Summary Action Arrives                              │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│  2. Generate Incremental Key                                │
│     Format: tenantId_campaignId_questionId_department       │
│             _aiTool_topic                                    │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│  3. Check State Store for Previous Summary                  │
└─────────────────────┬───────────────────────────────────────┘
                      │
        ┌─────────────┴─────────────┐
        │                           │
        ▼                           ▼
┌───────────────────┐    ┌──────────────────────┐
│  No Previous      │    │  Previous Summary    │
│  Summary Found    │    │  Found               │
└────────┬──────────┘    └──────────┬───────────┘
         │                          │
         ▼                          ▼
┌───────────────────┐    ┌──────────────────────┐
│  First-Time Path  │    │  Check Full Refresh  │
│  - Store current  │    │  Flag (LaunchDarkly) │
│  - Forward as-is  │    └──────────┬───────────┘
└───────────────────┘               │
                          ┌─────────┴──────────┐
                          │                    │
                          ▼                    ▼
                   ┌──────────────┐   ┌────────────────────┐
                   │ Full Refresh │   │ Incremental Update │
                   │ Enabled      │   │ - Merge current +  │
                   │ - Use First  │   │   previous         │
                   │   Time Path  │   │ - Call AI service  │
                   └──────────────┘   │ - Update store     │
                                      └────────────────────┘
```

### 3. Key Decision Logic

The processor decides whether to use incremental processing based on three conditions:

```java
if (previousSummaryAction == null
    || configReader.isDemoEnabled()
    || isFullRefreshSummarizationEnabled(currentSummaryAction.getTenantId())) {
    handleFirstTimeSummary(...);  // No incremental processing
} else {
    handleIncrementalUpdate(...);  // Use incremental processing
}
```

**Conditions for First-Time (Non-Incremental) Processing:**
1. **No Previous Summary**: First time processing for this campaign/dimension combination
2. **Demo Mode Enabled**: Configuration flag `demo.enabled=true`
3. **Full Refresh Flag Enabled**: LaunchDarkly flag `campaigns-enable-summarization-full-refresh` is true

---

## AI Drive vs DEX Campaigns

### AI Drive Campaigns

#### **Topology Path** (from `TopologyFactory.java`):
```
Source Stream (NQL Query Results)
    ↓
Filter: isAIDriveCampaign() 
    ↓
Group by Dimensions (AI Tool, Department, etc.)
    ↓
Summary Action Processor (SUMMARY_ACTION_PROCESSOR_AI_DRIVE)
    ↓
Incremental Summary Processor ← **USES INCREMENTAL SUMMARY**
    ↓
Output Topics (Summary & Insights)
```

#### **Key Characteristics:**
- ✅ **Uses Incremental Summary Processor**
- Processes comments grouped by dimensions (AI tool, department)
- Each dimension combination gets its own incremental key
- Example incremental key: `tenant123_campaign456_question789_Engineering_chatgpt_global`

#### **Prompts Used:**
- **Incremental**: `engage.euf-ai-free-text-summarizer/src/main/resources/prompts/ai-drive/incremental_summary_insights.txt`
- Focus: "AI tool usage and challenges" for end-users
- Context: Users work with AI copilot/assistant applications

### DEX Campaigns

#### **Topology Path** (from `TopologyFactory.java`):
```
Source Stream (NQL Query Results)
    ↓
Filter: isDexCampaign() (non-AI-Drive)
    ↓
Group Comments for Processing
    ↓
Summary Action Processor (SUMMARY_ACTION_PROCESSOR_DEX)
    ↓
Output Topics (Summary & Insights) ← **NO INCREMENTAL PROCESSOR**
```

#### **Key Characteristics:**
- ❌ **Does NOT use Incremental Summary Processor**
- Processes traditional DEX (Digital Experience) campaigns
- Each summary is generated fresh without previous context
- Simpler processing pipeline without state management

#### **Prompts Used:**
- **Standard**: Various prompts under `prompts/dex/` directory
- **Incremental Prompt Exists**: `prompts/dex/incremental_summary_insights.txt` (but not currently used in topology)
- Focus: "Employee experience" and "overall employee experience"

---

## Incremental Summary Implementation Details

### 1. Helper Class: `IncrementalSummaryActionFixedKeyProcessorHelper`

**Location**: `engage.euf-ai-free-text-summarizer/src/main/java/com/nexthink/euf/freetext/summarizer/kstream/processor/IncrementalSummaryActionFixedKeyProcessorHelper.java`

**Key Method**: `generateIncrementalSummaryAction()`

**Processing Steps:**
1. **Extract Request IDs**:
   - Current request IDs from new summary
   - Previous request IDs from stored summary (excluding duplicates)
   
2. **Merge Request IDs**:
   - Combines current and previous request IDs
   - Creates index mappings for tracking

3. **Prepare Summaries for AI**:
   - Formats both current and previous summaries
   - Creates structured input for AI service

4. **Invoke AI Service**:
   - Sends combined context to AI Gateway
   - Uses specialized incremental summary prompts

5. **Process AI Response**:
   - Parses AI response with incremental insights
   - Maps response back to request IDs
   - Updates telemetry data

6. **Update State Store**:
   - Stores trimmed version of new summary
   - Maintains state for next incremental update

### 2. Incremental Key Generation

The key format ensures unique storage for each dimension combination:

```
tenantId_campaignId_questionId_department_aiTool_topic
```

**Examples:**
- AI Drive ChatGPT: `itg123_campaign456_ai_usage_Engineering_chatgpt_global`
- AI Drive Claude: `itg123_campaign456_ai_usage_Marketing_claude_global`
- Null handling: `itg123_campaign456_ai_usage_null_null_global`

### 3. State Store Management

**Trimmed DTO** (`EngageCommentsSummaryActionDTOTrim`):
- Reduces memory footprint
- Stores only essential fields for comparison
- Enables efficient state store operations

---

## Configuration & Feature Flags

### LaunchDarkly Flags

#### 1. `campaigns-enable-summarization-full-refresh`
- **Default**: `false` (incremental enabled)
- **Purpose**: Override incremental processing
- **Scope**: Per-tenant configuration
- **Effect**: When `true`, forces first-time summary path (no incremental)

#### 2. `campaigns-enable-multi-page-summarization`
- **Default**: `true`
- **Purpose**: Enable/disable multi-page processing
- **Scope**: Per-tenant configuration

#### 3. `campaigns-generate-summary-of-topics-summaries`
- **Purpose**: Generate global summaries from topic summaries
- **Used in**: Summary action processing

### Application Configuration

**E2E Test Configuration** (`engage.euf-ai-e2e-tests/src/main/resources/application.yml`):

```yaml
engage-data-ingest:
  defaults:
    # AI-Drive specific configuration
    ai-drive-chatgpt-num-records: "${ENGAGE_DATA_INGEST_AI_DRIVE_CHATGPT_NUM_RECORDS:`2`}"
    ai-drive-claude-num-records: "${ENGAGE_DATA_INGEST_AI_DRIVE_CLAUDE_NUM_RECORDS:`2`}"
```

---

## Testing Considerations

### AI Drive E2E Tests

**Test File**: `engage.euf-ai-e2e-tests/src/main/java/com/nexthink/euf/ai/e2e/tests/tests/kafka/IntegrationTest.java`

**Phase 3 - AI Drive Testing**:
1. **Data Ingestion**: 
   - Configurable records per AI tool (default: 2 ChatGPT + 2 Claude = 4 total)
   - Can be overridden via environment variables

2. **Classifier Flow Verification**:
   - Validates analysis messages in `engage_campaign_comment-analysis` topic
   - Filters by AI-Drive campaign NQL ID and tenant ID

3. **Summarizer Flow Verification**:
   - Checks for summary messages in `engage_campaign_comments-summary-v2` topic
   - Validates insights in `engage_campaign_comments-generated-insights-v2` topic
   - Verifies incremental processing occurred

### Incremental Summary Testing Tips

**To Test Incremental Behavior**:
1. Run first ingestion → Check state store has entry
2. Run second ingestion → Verify incremental path is taken
3. Check logs for: "Stored incremental summary action for incrementalKey"
4. Verify AI telemetry shows `AI_OP_INCREMENTAL_SUMMARY` operation type

**To Force Full Refresh**:
1. Enable LaunchDarkly flag: `campaigns-enable-summarization-full-refresh=true`
2. Or enable demo mode in configuration
3. Check logs for: "First-time or full-refresh summary stored"

---

## Summary Comparison

| Feature | AI Drive | DEX |
|---------|----------|-----|
| **Incremental Summary** | ✅ Yes | ❌ No |
| **State Store Used** | `summary-action-incremental-store-v2` | None |
| **Dimension Grouping** | ✅ Yes (AI tool, department) | ❌ No |
| **Topology Processor** | `INCREMENTAL_SUMMARY_ACTION_PROCESSOR` | None |
| **Prompt Focus** | AI tool adoption & challenges | Employee experience |
| **State Management** | Complex (per dimension) | Simple (stateless) |
| **Cost Optimization** | Incremental AI calls | Full AI calls each time |
| **Context Continuity** | Maintained across runs | Fresh each time |

---

## Recommendations

### For AI Drive Testing:
1. **Default Configuration**: Use 2 ChatGPT + 2 Claude records for faster testing
2. **Incremental Testing**: Run multiple ingestion cycles to verify incremental behavior
3. **Monitor State Store**: Check logs for incremental key storage
4. **Dimension Testing**: Test different AI tool/department combinations

### For DEX Campaign Testing:
1. **Current Behavior**: No incremental processing (each run is independent)
2. **Potential Enhancement**: Could add incremental processing if needed
3. **Prompt Available**: `incremental_summary_insights.txt` exists but not wired in topology

### For Production:
1. **Monitor LaunchDarkly Flags**: Ensure proper tenant configuration
2. **State Store Maintenance**: Monitor size and performance
3. **Full Refresh Strategy**: Plan for when full refresh is needed (e.g., major campaign changes)

---

## Code References

### Key Files:
1. **Incremental Processor**: `IncrementalSummaryActionFixedKeyProcessor.java`
2. **Processor Helper**: `IncrementalSummaryActionFixedKeyProcessorHelper.java`
3. **Topology Factory**: `TopologyFactory.java` (lines 335-354 for AI Drive path)
4. **Constants**: `ProcessorConstants.java`, `Constants.java`
5. **AI Drive Prompts**: `prompts/ai-drive/incremental_summary_insights.txt`
6. **DEX Prompts**: `prompts/dex/incremental_summary_insights.txt` (exists but not used)

### State Store Constants:
- **Store Name**: `SUMMARY_ACTION_INCREMENTAL_STORE = "summary-action-incremental-store-v2"`
- **Processor Name**: `INCREMENTAL_SUMMARY_ACTION_PROCESSOR = "incremental-summary-action-processor"`
- **AI Operation**: `AI_OP_INCREMENTAL_SUMMARY = "incremental_summary_action"`

---

## Conclusion

**AI Drive campaigns use incremental summaries** to optimize costs and maintain context across multiple summary generations. The system stores previous summaries in a state store and uses them to generate incremental updates when new data arrives.

**DEX campaigns do NOT use incremental summaries** and generate fresh summaries each time, which is simpler but doesn't maintain historical context.

The incremental summary feature is controlled by:
1. Campaign type (AI Drive vs DEX)
2. LaunchDarkly feature flags (per-tenant)
3. Demo mode configuration

This architecture allows flexibility in choosing when to use incremental processing based on campaign requirements and tenant preferences.

