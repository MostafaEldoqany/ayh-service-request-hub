# Changelog

All notable changes to AYH Service Request Hub will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] - 2025-09-25

### Added
- Initial project structure and workspace setup
- Environment configuration template (.env.example)
- Development logging framework
- Git repository initialization
- Conventional commit standards

### Project Structure
```
ayh-service-request-hub/
├── solution/           # Exported Power Platform solution files
├── dataverse/schema/   # Dataverse table definitions and seed data
├── powerapps/canvas/   # Canvas app exports (.msapp files)
├── flows/             # Power Automate flow definitions
├── powerbi/           # Power BI dashboard files (.pbix)
├── env/               # Environment-specific configuration
├── logs/              # Build and deployment logs
└── docs/              # Documentation and change management
```

### Development Notes
- Build agent: Autonomous Power Platform solution construction
- Target environment: Microsoft Dataverse + Power Apps + Power Automate + Power BI
- Security model: 4-tier role-based access (Staff, Agent, Manager, Admin)
- Integration: Teams adaptive cards, AAD groups, SLA automation