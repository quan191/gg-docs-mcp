# Google Docs MCP Server

An MCP (Model Context Protocol) server that enables integration with Google Docs API for document management and editing capabilities.

## 🏗️ Architecture Overview

### Technology Stack
- **Language**: TypeScript
- **Authentication**: Google Service Account
- **APIs**: Google Docs API v1, Google Drive API v3
- **MCP SDK**: @modelcontextprotocol/sdk

### System Architecture

```
┌─────────────────────────────────────────┐
│         MCP Clients                     │
│  (Claude Code, Cursor, Terminal)        │
└────────────┬────────────────────────────┘
             │ stdio/SSE
┌────────────▼────────────────────────────┐
│      MCP Server (TypeScript)            │
│  ┌────────────────────────────────┐     │
│  │   MCP Protocol Handler         │     │
│  └────────────────────────────────┘     │
│  ┌────────────────────────────────┐     │
│  │   Google Docs Service Layer    │     │
│  └────────────────────────────────┘     │
│  ┌────────────────────────────────┐     │
│  │   Authentication Manager       │     │
│  │   (Service Account)            │     │
│  └────────────────────────────────┘     │
└────────────┬────────────────────────────┘
             │ HTTPS
┌────────────▼────────────────────────────┐
│        Google Docs API v1               │
│        Google Drive API v3              │
└─────────────────────────────────────────┘
```

## 📋 Features

### Core Capabilities
- List documents from My Drive, Shared Drives, and Shared with Me
- Create new documents in specified locations
- Get document content and metadata
- Modify documents with formatting support
- Search documents by title or content

### MCP Tools

#### 1. `list_documents`
- List docs from My Drive, Shared Drives, and Shared with Me
- Filter by folder, owner, or last modified
- Support pagination for large document sets

#### 2. `create_document`
- Create in My Drive or specific Shared Drive
- Set initial title and content
- Configure sharing permissions

#### 3. `get_document`
- Fetch full document content and structure
- Return formatted text with metadata
- Support for different content representations

#### 4. `modify_document`
- **Smart Page Management**: 
  - Append content to new page (avoiding conflicts)
  - Modify specific sections by heading/position
- **Formatting Support**:
  - Headings (H1-H6)
  - Text styling (bold, italic, underline)
  - Lists (bullet and numbered)
  - Tables
  - Links
- Batch operations for efficiency

#### 5. `search_documents`
- Search by title or content keywords
- Filter by document type and location

## 🚀 Document Modification Strategy

To avoid conflicts with concurrent editors:

```typescript
// Approach 1: Append to end with page break
appendToDocument(docId, content, { addPageBreak: true })

// Approach 2: Section-based editing
modifySection(docId, { 
  sectionIdentifier: "## My Section", 
  content: newContent 
})

// Approach 3: Get content, modify locally, update specific range
const doc = getDocument(docId)
updateRange(docId, startIndex, endIndex, newContent)
```

## 📁 Project Structure

```
google-docs-mcp/
├── src/
│   ├── index.ts           # MCP server entry
│   ├── auth/
│   │   └── service-account.ts
│   ├── services/
│   │   ├── docs-service.ts
│   │   └── drive-service.ts
│   ├── tools/
│   │   ├── list-documents.ts
│   │   ├── create-document.ts
│   │   ├── get-document.ts
│   │   └── modify-document.ts
│   └── types/
├── config/
│   └── service-account.json
├── package.json
└── README.md
```

## 🗓️ Implementation Roadmap

### Phase 1: Foundation (Week 1)
- [ ] Set up TypeScript MCP server boilerplate
- [ ] Configure Service Account authentication
- [ ] Implement basic Drive API connection
- [ ] Create development environment setup

### Phase 2: Core Features (Week 2)
- [ ] Implement list documents from all sources
- [ ] Create new documents functionality
- [ ] Get document content with proper formatting
- [ ] Basic error handling

### Phase 3: Advanced Editing (Week 3)
- [ ] Text formatting (bold, italic, headings)
- [ ] Table operations
- [ ] Link insertion
- [ ] Smart page/section management
- [ ] Batch operations support

### Phase 4: Polish (Week 4)
- [ ] Comprehensive error handling and retries
- [ ] Caching for performance optimization
- [ ] Documentation and examples
- [ ] Testing suite

## 🔐 Authentication Setup

### Service Account Configuration
1. Create a project in Google Cloud Console
2. Enable Google Docs API and Google Drive API
3. Create a Service Account
4. Download the JSON key file
5. Share documents/drives with the service account email
6. Store credentials securely in `config/service-account.json`

## ⚙️ Configuration

### Environment Variables
```env
GOOGLE_SERVICE_ACCOUNT_PATH=./config/service-account.json
MCP_SERVER_PORT=3000
LOG_LEVEL=info
```

### MCP Client Configuration
```json
{
  "mcpServers": {
    "google-docs": {
      "command": "node",
      "args": ["./dist/index.js"],
      "env": {
        "GOOGLE_SERVICE_ACCOUNT_PATH": "./config/service-account.json"
      }
    }
  }
}
```

## 🚨 Key Considerations

1. **Permissions**: Service account needs to be added as editor to Shared Drives
2. **Rate Limits**: Google API has quotas (300 requests/minute)
3. **Conflict Avoidance**: Always append or use designated sections when multiple users edit
4. **Performance**: Cache document structure to minimize API calls
5. **Security**: Never commit service account credentials to version control

## 📝 Usage Examples

### List All Documents
```typescript
// MCP Request
{
  "tool": "list_documents",
  "arguments": {
    "source": "all",
    "limit": 10
  }
}
```

### Create New Document
```typescript
// MCP Request
{
  "tool": "create_document",
  "arguments": {
    "title": "Meeting Notes",
    "content": "# Meeting Notes\n\n## Agenda\n- Item 1\n- Item 2",
    "location": "shared_drive_id"
  }
}
```

### Modify Document
```typescript
// MCP Request
{
  "tool": "modify_document",
  "arguments": {
    "documentId": "doc_id_here",
    "operations": [
      {
        "type": "appendPage",
        "content": "## New Section\nContent here..."
      }
    ]
  }
}
```

## 🛠️ Development

### Prerequisites
- Node.js 18+
- TypeScript 5+
- Google Cloud Project with APIs enabled
- Service Account credentials

### Installation
```bash
npm install
npm run build
npm run dev
```

### Testing
```bash
npm test
```

## 📚 Resources

- [MCP Documentation](https://modelcontextprotocol.io/)
- [Google Docs API Reference](https://developers.google.com/docs/api)
- [Google Drive API Reference](https://developers.google.com/drive/api)
- [Service Account Setup Guide](https://cloud.google.com/iam/docs/service-accounts)

## 📄 License

MIT