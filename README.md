# AdOps-Airtable

Airtable API automation tools and utilities for Evolution Media Group.

## Overview

AdOps-Airtable is part of the AdOps automation platform family, providing centralized Airtable-focused automation tools, utilities, and reusable components.

## Features

- **Airtable API Utilities** - Python and JavaScript tools for Airtable REST API
- **Shared Automation Libraries** - Reusable components for Airtable scripting
- **Cross-Base Synchronization** - Tools for syncing data between Airtable bases
- **Automation Templates** - Production-ready script templates

## Related Projects

| Project | Platform | Description |
|---------|----------|-------------|
| [AdOps-TTD](https://github.com/evolution-media-group/AdOps-TTD) | The Trade Desk | Creative, audience, and campaign automation |
| [AdOps-SFC](https://github.com/evolution-media-group/AdOps-SFC) | Salesforce | API integration tools |
| [AdOps-Camphouse](https://github.com/evolution-media-group/AdOps-Camphouse) | Camphouse | Platform automation |

## Project Structure

```
AdOps-Airtable/
├── docs/                    # Documentation
│   ├── prompts/             # Development history
│   ├── api/                 # API documentation
│   ├── architecture/        # System design docs
│   └── patterns/            # Shared patterns
├── scripts/                 # Automation scripts
│   ├── airtable/current/    # Production scripts
│   ├── deployment/          # Deployment utilities
│   └── maintenance/         # Maintenance scripts
├── utility/                 # Tools and utilities
│   ├── testing/             # Test scripts
│   ├── analysis/            # Analysis tools
│   └── helpers/             # Helper functions
├── deploy/                  # Deployment configs
├── data/                    # Schemas and samples
├── config/                  # Configuration files
└── shared/                  # Shared resources
```

## Quick Start

### Prerequisites

- Python 3.7+
- Node.js 14+ (for Airtable script validation)
- Airtable API access

### Installation

```bash
# Clone the repository
git clone https://github.com/evolution-media-group/AdOps-Airtable.git
cd AdOps-Airtable

# Install Python dependencies (when available)
pip install -r requirements.txt
```

## Development

### Airtable Script Standards

All Airtable automation scripts follow these conventions:

1. **Version tracking** in header comments and constants
2. **Single `input.config()` call** at script initialization
3. **Comprehensive logging** with API request/response tracking
4. **Error handling** with proper re-throw for automation handling

See [CLAUDE.md](CLAUDE.md) for complete development conventions.

### Git Workflow

```bash
git add -A
git commit -m "type: description (v1.2.3)

- Specific change 1
- Specific change 2
- Why: Business/technical reason

🤖 Generated with [Claude Code](https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>"
git push
```

## Documentation

- [CLAUDE.md](CLAUDE.md) - Development configuration and standards
- [docs/prompts/](docs/prompts/) - Development history
- [docs/patterns/](docs/patterns/) - Shared implementation patterns

## Contributing

Please follow existing code patterns and include documentation for any new features.

## License

Property of Evolution Media Group

## Support

For issues or questions, contact the EMG development team.
