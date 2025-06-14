# Distill - AI-Powered Document & Video Processing Platform

🚀 **Live Demo**: [https://distill-frontend-dj2j.onrender.com/](https://distill-frontend-dj2j.onrender.com/)

Distill is a modern web application that allows users to upload documents (PDFs) and YouTube videos, process them using AI, and interact with the content through an intelligent chat interface. The platform leverages advanced language models and vector embeddings to provide contextual answers based on your uploaded content.

## ✨ Features

### 📄 Document Processing
- **PDF Upload & Processing**: Upload PDF documents and extract text content
- **Intelligent Chunking**: Automatically splits documents into meaningful chunks for better processing
- **Vector Embeddings**: Converts content into searchable vector embeddings using Cohere AI

### 🎥 YouTube Video Processing
- **URL-based Upload**: Simply paste a YouTube URL to process video content
- **Transcript Extraction**: Automatically extracts and processes video transcripts
- **Multi-language Support**: Supports transcripts in multiple languages with fallback options
- **Rate Limiting Protection**: Built-in retry logic and cookie authentication for reliable processing

### 💬 AI Chat Interface
- **Contextual Conversations**: Chat with your uploaded content using advanced AI models
- **Real-time Responses**: Powered by Groq's fast inference API
- **Session Management**: Maintains conversation history and context
- **Smart Retrieval**: Uses Pinecone vector database for accurate content retrieval

### 🔐 User Authentication
- **Secure Registration & Login**: JWT-based authentication system
- **User Namespaces**: Each user's content is isolated and secure
- **Session Management**: Persistent login sessions with secure token handling

### 🎨 Modern UI/UX
- **Responsive Design**: Works seamlessly on desktop and mobile devices
- **Real-time Feedback**: Loading states, progress indicators, and toast notifications
- **Dark/Light Theme**: Modern interface built with Tailwind CSS
- **Smooth Animations**: Enhanced user experience with Framer Motion

## 🛠️ Tech Stack

### Frontend
- **React 19** - Modern React with latest features
- **Vite** - Fast build tool and development server
- **Tailwind CSS** - Utility-first CSS framework
- **React Router** - Client-side routing
- **Framer Motion** - Smooth animations and transitions
- **Lucide React** - Beautiful icon library
- **React Hot Toast** - Elegant notifications

### Backend
- **FastAPI** - Modern Python web framework
- **Python 3.11+** - Latest Python features
- **Uvicorn** - ASGI server for production
- **PostgreSQL** - Robust relational database
- **Asyncpg** - Async PostgreSQL driver

### AI & ML Services
- **Cohere AI** - Text embeddings and language processing
- **Groq** - Fast LLM inference for chat responses
- **Pinecone** - Vector database for semantic search
- **OpenAI-compatible APIs** - Flexible LLM integration

### Content Processing
- **yt-dlp** - YouTube video downloading and metadata extraction
- **youtube-transcript-api** - YouTube transcript extraction
- **PyPDF2 & PyMuPDF** - PDF text extraction and processing
- **Custom Chunking Algorithm** - Intelligent text segmentation

### Authentication & Security
- **JWT Tokens** - Secure authentication
- **Passlib + Bcrypt** - Password hashing
- **CORS Middleware** - Cross-origin request handling
- **Environment Variables** - Secure configuration management

## 🚀 Quick Start

### Prerequisites
- **Node.js 18+** and npm
- **Python 3.11+** and pip
- **PostgreSQL** database
- API keys for: Cohere, Groq, and Pinecone

### 1. Clone the Repository
```bash
git clone <repository-url>
cd distill
```

### 2. Backend Setup
```bash
cd Backend
pip install -r requirements.txt

# Create .env file with your API keys
cp .env.example .env
# Edit .env with your actual API keys and database URL
```

### 3. Frontend Setup
```bash
cd Frontend
npm install
```

### 4. Database Setup
```bash
# Run database migrations (if any)
python Backend/Database/setup.py
```

### 5. Start Development Servers

**Backend** (from Backend directory):
```bash
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

**Frontend** (from Frontend directory):
```bash
npm run dev
```

Visit `http://localhost:5173` to access the application.

## 🔧 Configuration

### Environment Variables

Create a `.env` file in the Backend directory:

```env
# Database
DATABASE_URL=postgresql://username:password@localhost:5432/distill

# AI Services
COHERE_API_KEY=your_cohere_api_key
GROQ_API_KEY=your_groq_api_key
PINECONE_API_KEY=your_pinecone_api_key
PINECONE_ENVIRONMENT=your_pinecone_environment

# Authentication
JWT_SECRET_KEY=your_jwt_secret_key
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

# Optional: YouTube Cookies (for rate limiting)
YOUTUBE_COOKIES_PATH=./youtube_cookies.txt
```

### YouTube Cookies (Optional)
For better YouTube processing reliability, you can provide cookies:
1. Export cookies from your browser using a browser extension
2. Save as `youtube_cookies.txt` in the Backend directory
3. The system will automatically detect and use them

## 📁 Project Structure

```
distill/
├── Frontend/                 # React frontend application
│   ├── src/
│   │   ├── components/      # Reusable UI components
│   │   ├── pages/          # Page components
│   │   ├── hooks/          # Custom React hooks
│   │   └── utils/          # Utility functions
│   ├── public/             # Static assets
│   └── package.json        # Frontend dependencies
├── Backend/                 # FastAPI backend application
│   ├── routes/             # API route handlers
│   ├── auth/               # Authentication logic
│   ├── Database/           # Database models and migrations
│   ├── Processing/         # Content processing utilities
│   ├── Ingestion/          # File and video ingestion
│   ├── LLM/                # Language model integrations
│   ├── Parsed_files/       # Processed content storage
│   └── main.py             # FastAPI application entry point
├── main.py                 # Root application entry point
├── requirements.txt        # Python dependencies
└── README.md              # This file
```

## 🔄 API Endpoints

### Authentication
- `POST /auth/register` - User registration
- `POST /auth/login` - User login
- `GET /auth/me` - Get current user info

### Content Processing
- `POST /upload/pdf` - Upload and process PDF documents
- `POST /upload/youtube` - Process YouTube videos
- `GET /files` - List user's uploaded files

### Chat Interface
- `POST /chat` - Send chat messages and get AI responses
- `GET /chat/history` - Retrieve chat history

### Health & Status
- `GET /health` - Application health check
- `GET /` - API status and information

## 🚀 Deployment

The application is deployed on Render with the following configuration:

### Production URLs
- **Frontend**: [https://distill-frontend-dj2j.onrender.com/](https://distill-frontend-dj2j.onrender.com/)
- **Backend API**: Automatically configured for production

### Deployment Features
- **Automatic Deployments** from main branch
- **Environment Variable Management** through Render dashboard
- **PostgreSQL Database** hosted on Render
- **Static File Serving** for frontend assets
- **HTTPS/SSL** enabled by default

### Production Optimizations
- **Absolute Path Resolution** for reliable file handling
- **Comprehensive Error Handling** with detailed logging
- **Rate Limiting Protection** for external APIs
- **Database Connection Pooling** for better performance
- **CORS Configuration** for secure cross-origin requests

## 🐛 Debugging & Monitoring

The application includes comprehensive debug logging:

### Frontend Logging
- `🎥 [YT-DEBUG]` - YouTube processing steps
- `🔄 [EMBED-DEBUG]` - Embedding generation
- `🎬 [YT-FLOW]` - Complete YouTube workflow

### Backend Logging
- `🎥 [YT-BACKEND]` - YouTube processing backend
- `🎬 [YT-HANDLER]` - YouTube handler operations
- `🔄 [EMBED-BACKEND]` - Embedding processing

### Error Handling
- Detailed error messages for different failure scenarios
- Automatic retry logic for transient failures
- Graceful degradation for service unavailability

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🆘 Support

If you encounter any issues or have questions:

1. Check the [Issues](../../issues) section
2. Review the debug logs in the browser console
3. Ensure all environment variables are properly configured
4. Verify API keys and service availability

## 🔮 Roadmap

- [ ] Support for more document formats (Word, PowerPoint, etc.)
- [ ] Batch processing for multiple files
- [ ] Advanced search and filtering
- [ ] Export chat conversations
- [ ] Mobile app development
- [ ] Integration with more AI models
- [ ] Real-time collaboration features

---

**Built with ❤️ using modern web technologies**