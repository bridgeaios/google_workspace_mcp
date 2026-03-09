# Google Workspace MCP Server Configuration Guide

## Overview

This document describes the configuration files and environment variables for the Google Workspace MCP Server with OAuth 2.1 support.

## Configuration Files

### 1. Environment Configuration (`.env`)

The `.env` file contains all environment variables for the server. Key sections:

#### Required Settings

| Variable | Description | Required |
|----------|-------------|----------|
| `GOOGLE_OAUTH_CLIENT_ID` | Google OAuth 2.0 Client ID | Yes |
| `GOOGLE_OAUTH_CLIENT_SECRET` | Google OAuth 2.0 Client Secret | Yes |

#### OAuth 2.1 Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `MCP_ENABLE_OAUTH21` | `true` | Enable OAuth 2.1 multi-user support |
| `EXTERNAL_OAUTH21_PROVIDER` | `false` | Use external OAuth provider |
| `WORKSPACE_MCP_STATELESS_MODE` | `true` | Stateless operation (container-friendly) |
| `OAUTH2_ENABLE_LEGACY_AUTH` | `true` | Keep legacy auth for compatibility |

#### Server Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `WORKSPACE_MCP_PORT` | `8000` | Server port |
| `WORKSPACE_MCP_HOST` | `0.0.0.0` | Bind host |
| `WORKSPACE_MCP_BASE_URI` | `http://localhost` | Base URI |

#### Storage Backend

| Variable | Default | Description |
|----------|---------|-------------|
| `WORKSPACE_MCP_OAUTH_PROXY_STORAGE_BACKEND` | `memory` | Storage: `memory`, `disk`, or `valkey` |

### 2. Tool Tier Configuration (`core/tool_tiers.yaml`)

Tools are organized into three tiers per Google Workspace service:

#### Gmail Tools

**Core (5 tools):**
- `search_gmail_messages` - Search emails
- `get_gmail_message_content` - Read single email
- `get_gmail_messages_content_batch` - Batch read emails
- `send_gmail_message` - Send email

**Extended (8 tools):**
- `get_gmail_attachment_content` - Download attachments
- `get_gmail_thread_content` - Read thread
- `modify_gmail_message_labels` - Manage labels
- `list_gmail_labels` - List labels
- `manage_gmail_label` - Create/delete labels
- `draft_gmail_message` - Create drafts
- `list_gmail_filters` - List filters
- `create_gmail_filter` - Create filters
- `delete_gmail_filter` - Delete filters

**Complete (3 tools):**
- `get_gmail_threads_content_batch` - Batch thread operations
- `batch_modify_gmail_message_labels` - Batch label changes
- `start_google_auth` - Authentication helper

#### Drive Tools

**Core (7 tools):**
- `search_drive_files` - Search files
- `get_drive_file_content` - Read files
- `get_drive_file_download_url` - Get download links
- `create_drive_file` - Create files
- `create_drive_folder` - Create folders
- `import_to_google_doc` - Import documents
- `share_drive_file` - Share files
- `get_drive_shareable_link` - Generate share links

**Extended (7 tools):**
- `list_drive_items` - List directory contents
- `copy_drive_file` - Copy files
- `update_drive_file` - Update metadata
- `update_drive_permission` - Update permissions
- `remove_drive_permission` - Remove access
- `transfer_drive_ownership` - Transfer ownership
- `batch_share_drive_file` - Batch sharing
- `set_drive_file_permissions` - Set permissions

**Complete (2 tools):**
- `get_drive_file_permissions` - List permissions
- `check_drive_file_public_access` - Check public access

#### Calendar Tools

**Core (4 tools):**
- `list_calendars` - List calendars
- `get_events` - Get events
- `create_event` - Create events
- `modify_event` - Update events

**Extended (2 tools):**
- `delete_event` - Delete events
- `query_freebusy` - Check availability

**Complete:** None

#### Docs Tools

**Core (3 tools):**
- `get_doc_content` - Read documents
- `create_doc` - Create documents
- `modify_doc_text` - Edit text

**Extended (6 tools):**
- `export_doc_to_pdf` - Export to PDF
- `search_docs` - Search documents
- `find_and_replace_doc` - Find/replace
- `list_docs_in_folder` - List in folder
- `insert_doc_elements` - Insert elements
- `update_paragraph_style` - Format paragraphs
- `get_doc_as_markdown` - Export as Markdown

**Complete (10 tools):**
- `insert_doc_image` - Insert images
- `update_doc_headers_footers` - Headers/footers
- `batch_update_doc` - Batch operations
- `inspect_doc_structure` - Structure analysis
- `create_table_with_data` - Create tables
- `debug_table_structure` - Debug tables
- `read_document_comments` - Read comments
- `create_document_comment` - Add comments
- `reply_to_document_comment` - Reply to comments
- `resolve_document_comment` - Resolve comments

#### Sheets Tools

**Core (3 tools):**
- `create_spreadsheet` - Create spreadsheets
- `read_sheet_values` - Read data
- `modify_sheet_values` - Write data

**Extended (3 tools):**
- `list_spreadsheets` - List files
- `get_spreadsheet_info` - Get metadata
- `format_sheet_range` - Format cells

**Complete (4 tools):**
- `create_sheet` - Create worksheets
- `read_spreadsheet_comments` - Read comments
- `create_spreadsheet_comment` - Add comments
- `reply_to_spreadsheet_comment` - Reply to comments
- `resolve_spreadsheet_comment` - Resolve comments

#### Chat Tools

**Core (4 tools):**
- `send_message` - Send messages
- `get_messages` - Read messages
- `search_messages` - Search messages
- `create_reaction` - Add reactions

**Extended (2 tools):**
- `list_spaces` - List spaces
- `download_chat_attachment` - Download files

**Complete:** None

#### Forms Tools

**Core (2 tools):**
- `create_form` - Create forms
- `get_form` - Get form details

**Extended (1 tool):**
- `list_form_responses` - List responses

**Complete (3 tools):**
- `set_publish_settings` - Publish settings
- `get_form_response` - Get single response
- `batch_update_form` - Batch updates

#### Slides Tools

**Core (2 tools):**
- `create_presentation` - Create presentations
- `get_presentation` - Get presentation

**Extended (3 tools):**
- `batch_update_presentation` - Batch updates
- `get_page` - Get slide
- `get_page_thumbnail` - Get thumbnail

**Complete (4 tools):**
- `read_presentation_comments` - Read comments
- `create_presentation_comment` - Add comments
- `reply_to_presentation_comment` - Reply to comments
- `resolve_presentation_comment` - Resolve comments

#### Tasks Tools

**Core (4 tools):**
- `get_task` - Get task
- `list_tasks` - List tasks
- `create_task` - Create task
- `update_task` - Update task

**Extended (1 tool):**
- `delete_task` - Delete task

**Complete (6 tools):**
- `list_task_lists` - List task lists
- `get_task_list` - Get task list
- `create_task_list` - Create task list
- `update_task_list` - Update task list
- `delete_task_list` - Delete task list
- `move_task` - Move task
- `clear_completed_tasks` - Clear completed

#### Contacts Tools

**Core (4 tools):**
- `search_contacts` - Search contacts
- `get_contact` - Get contact
- `list_contacts` - List contacts
- `create_contact` - Create contact

**Extended (4 tools):**
- `update_contact` - Update contact
- `delete_contact` - Delete contact
- `list_contact_groups` - List groups
- `get_contact_group` - Get group

**Complete (7 tools):**
- `batch_create_contacts` - Batch create
- `batch_update_contacts` - Batch update
- `batch_delete_contacts` - Batch delete
- `create_contact_group` - Create group
- `update_contact_group` - Update group
- `delete_contact_group` - Delete group
- `modify_contact_group_members` - Manage members

#### Search Tools

**Core (1 tool):**
- `search_custom` - Custom search (requires API key)

**Extended (1 tool):**
- `search_custom_siterestrict` - Site-restricted search

**Complete (1 tool):**
- `get_search_engine_info` - Get search engine info

#### Apps Script Tools

**Core (7 tools):**
- `list_script_projects` - List projects
- `get_script_project` - Get project
- `get_script_content` - Get code
- `create_script_project` - Create project
- `update_script_content` - Update code
- `run_script_function` - Run function
- `generate_trigger_code` - Generate triggers

**Extended (9 tools):**
- `create_deployment` - Create deployment
- `list_deployments` - List deployments
- `update_deployment` - Update deployment
- `delete_deployment` - Delete deployment
- `delete_script_project` - Delete project
- `list_versions` - List versions
- `create_version` - Create version
- `get_version` - Get version
- `list_script_processes` - List processes
- `get_script_metrics` - Get metrics

**Complete:** None

## Tool Tier Selection

Set the `TOOL_TIER` environment variable to control which tools are available:

- **`core`** - Essential tools only (read, search, basic create/send)
- **`extended`** - Core + advanced operations (batch, formatting, comments)
- **`complete`** - All available tools including administrative functions

### Default Tool Counts by Tier

| Service | Core | Extended | Complete |
|---------|------|----------|----------|
| Gmail | 5 | 13 | 16 |
| Drive | 7 | 14 | 16 |
| Calendar | 4 | 6 | 6 |
| Docs | 3 | 9 | 19 |
| Sheets | 3 | 6 | 10 |
| Chat | 4 | 6 | 6 |
| Forms | 2 | 3 | 6 |
| Slides | 2 | 5 | 9 |
| Tasks | 4 | 5 | 11 |
| Contacts | 4 | 8 | 15 |
| Search | 1 | 2 | 3 |
| Apps Script | 7 | 16 | 16 |
| **Total** | **46** | **93** | **137** |

## Docker Compose Configuration

The `docker-compose.yml` file configures the container with:

- Port mapping: `8000:8000`
- Volume mounts for credentials
- Environment file injection from `.env`
- Stateful credential storage via `store_creds` volume

## Next Steps

1. **Configure OAuth Credentials:**
   - Go to [Google Cloud Console](https://console.cloud.google.com/)
   - Create OAuth 2.0 Desktop Application credentials
   - Update `GOOGLE_OAUTH_CLIENT_ID` and `GOOGLE_OAUTH_CLIENT_SECRET` in `.env`

2. **Select Tool Tier:**
   - Set `TOOL_TIER=core` for minimal deployment
   - Set `TOOL_TIER=extended` for full functionality
   - Set `TOOL_TIER=complete` for administrative access

3. **Start the Server:**
   ```bash
   # Using Docker Compose
   docker-compose up -d
   
   # Or using Python directly
   uv run python main.py --transport streamable-http
   ```

4. **Verify Configuration:**
   - Check logs for successful startup
   - Verify OAuth 2.1 endpoints are accessible
   - Test tool availability via MCP inspector

## Security Notes

- Never commit `.env` files with real credentials
- Use `WORKSPACE_MCP_STATELESS_MODE=true` for container deployments
- Enable HTTPS in production (disable `OAUTHLIB_INSECURE_TRANSPORT`)
- Use Valkey/Redis for distributed deployments
- Set `FASTMCP_SERVER_AUTH_GOOGLE_JWT_SIGNING_KEY` for stable encryption
