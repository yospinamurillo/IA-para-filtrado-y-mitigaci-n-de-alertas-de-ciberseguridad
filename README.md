# IA-para-filtrado-y-mitigaci-n-de-alertas-de-ciberseguridad
A para filtrado y mitigación de alertas de ciberseguridad
# Cybersecurity Alert Automation System

**AI-powered cybersecurity alert filtering, classification, and automated mitigation**

A comprehensive SOAR (Security Orchestration, Automation, and Response) system that uses machine learning to automatically process, prioritize, and respond to security alerts.

## Features

### Core Capabilities
- **AI-Powered Alert Classification** - Automatically categorize and score alerts using ML
- **Automated Rule Engine** - Apply filtering, escalation, and suppression rules
- **Mitigation Playbooks** - Predefined response workflows for common threats
- **Real-time Processing** - Ingest and process thousands of alerts per minute
- **Enrichment Integration** - GeoIP, threat intelligence, asset context
- **Audit Trail** - Complete logging of all actions

### Alert Sources
- SIEM integrations (Splunk, QRadar, ELK)
- Firewall logs (Palo Alto, Cisco, etc.)
- IDS/IPS (Snort, Suricata)
- Antivirus/EDR solutions
- Custom API ingestion
- Syslog, JSON, CSV formats

### AI Analysis
- Machine learning classification (Random Forest)
- False positive detection
- Risk scoring (0-1 scale)
- Priority recommendations
- Behavioral baselines

### Automated Mitigation
- IP blocking (firewall integration)
- Host isolation (network controls)
- User account actions
- Quarantine operations
- Custom playbook execution

### Integrations
- Firewalls (Palo Alto, Cisco, iptables)
- SIEM systems
- EDR/Antivirus solutions
- Email notifications (SMTP)
- Threat intelligence feeds
- REST API for custom connectors

## Technology Stack

- **Backend**: Python 3.11, FastAPI
- **Database**: PostgreSQL (alerts, rules, audit)
- **Cache/Queue**: Redis + Celery
- **ML**: scikit-learn, pandas, numpy
- **ORM**: SQLAlchemy
- **Auth**: JWT tokens
- **Container**: Docker, docker-compose

## Quick Start

### Prerequisites
```bash
# Ensure you have these installed:
- Docker & Docker Compose
- Python 3.11+ (for local development)
- Git
```

### 1. Clone and Setup
```bash
git clone <repository>
cd cybersecurity-alert-automation
cp .env.example .env
# Edit .env with your configuration
```

### 2. Using Docker Compose (Recommended)
```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f app

# Stop services
docker-compose down
```

Services will be available at:
- API: http://localhost:8000
- API Docs: http://localhost:8000/docs
- Celery Flower (monitoring): http://localhost:5555 (if monitoring profile enabled)

### 3. Manual Installation

```bash
# Install dependencies
pip install -r requirements.txt

# Setup database
createdb cybersecurity_db
export DATABASE_URL=postgresql://user:password@localhost/cybersecurity_db

# Run migrations
alembic upgrade head

# Start Redis
redis-server

# Start Celery worker (optional, for async tasks)
celery -A app.celery_app worker --loglevel=info

# Start API server
uvicorn app.main:app --reload
```

## Configuration

Edit `.env` file or set environment variables:

```bash
# Required
DATABASE_URL=postgresql://user:password@localhost/cybersecurity_db
REDIS_URL=redis://localhost:6379/0
SECRET_KEY=your-secret-key

# Optional
DEBUG=True
AUTO_MITIGATE_THRESHOLD=0.85  # Auto-mitigate alerts with AI score above this
ALERT_RETENTION_DAYS=90
MAX_ALERTS_PER_MINUTE=1000
```

## API Usage

### Ingest an Alert
```bash
curl -X POST "http://localhost:8000/api/alerts" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Suspicious login from unknown IP",
    "description": "Multiple failed login attempts detected",
    "source": "siem",
    "source_type": "splunk",
    "severity": "high",
    "ip_address": "192.168.1.100",
    "hostname": "web-server-01"
  }'
```

### List Alerts
```bash
# Get all alerts
curl "http://localhost:8000/api/alerts"

# Filter by severity
curl "http://localhost:8000/api/alerts?severity=critical"

# Paginate
curl "http://localhost:8000/api/alerts?page=1&page_size=50"
```

### Get Statistics
```bash
curl "http://localhost:8000/api/stats/overview"
curl "http://localhost:8000/api/stats/alerts"
```

### Authentication
```bash
# Login
curl -X POST "http://localhost:8000/api/auth/token" \
  -d "username=admin&password=admin123"

# Use token
curl "http://localhost:8000/api/alerts" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

## Creating Rules

Rules define automated actions based on conditions.

### Example: Block High-Severity External IPs
```json
{
  "name": "Block Critical External IPs",
  "type": "mitigate",
  "priority": 10,
  "conditions": [
    {
      "field": "severity",
      "operator": "eq",
      "value": "critical"
    },
    {
      "field": "ip_address",
      "operator": "contains",
      "value": "external"
    }
  ],
  "action_config": {
    "action_type": "block_ip",
    "parameters": {"duration": 86400}
  }
}
```

### Rule Types
- **filter** - Suppress alerts
- **escalate** - Increase severity
- **suppress** - Mark as false positive
- **enrich** - Add context data
- **mitigate** - Automatic response
- **notify** - Send alerts

### Condition Operators
- `eq` - Equals
- `ne` - Not equals
- `contains` - Contains text
- `gt` - Greater than
- `lt` - Less than
- `in` - In list
- `regex` - Regular expression

## Playbooks

Predefined sequences of mitigation actions.

### Example Playbook for Malware
```json
{
  "name": "Malware Response Playbook",
  "category": "malware",
  "steps": [
    {
      "action_type": "isolate_host",
      "parameters": {"duration": 3600},
      "description": "Isolate infected host"
    },
    {
      "action_type": "quarantine_host",
      "parameters": {"scan_type": "full"},
      "description": "Initiate full AV scan",
      "condition": {"field": "ai_score", "operator": "gt", "value": 0.7}
    }
  ]
}
```

## Database Schema

Main tables:
- `alerts` - Security incidents
- `alert_enrichments` - Context data
- `rules` - Automation rules
- `rule_executions` - Rule execution logs
- `mitigation_actions` - Mitigation history
- `playbooks` - Response workflows
- `users` - System users
- `audit_logs` - Audit trail

## ML Model

The system uses machine learning for alert classification:

### Training
```python
from app.ai.processor import ai_processor

# Train on historical data
ai_processor.train_from_historical(db_session)

# Save model
ai_processor.classifier.save_model('data/models/alert_classifier.pkl')
```

### Feature Extraction
- Text features (TF-IDF from title/description)
- Severity encoding
- Network context (IP, port)
- Time-based features
- Keyword matching

### Categories
- malware
- phishing
- ddos
- intrusion
- data_exfiltration
- privilege_escalation
- lateral_movement
- suspicious_activity
- compliance

## Development

### Project Structure
```
app/
├── main.py              # FastAPI application entry
├── config.py            # Configuration
├── database.py          # DB connection
├── models/              # SQLAlchemy models
│   ├── alert.py
│   ├── rule.py
│   ├── mitigation.py
│   ├── user.py
│   └── audit.py
├── schemas/             # Pydantic schemas
├── api/                 # API routes
│   ├── alerts.py
│   ├── rules.py
│   ├── mitigation.py
│   ├── auth.py
│   └── stats.py
├── services/            # Business logic
│   ├── alert.py
│   └── rule_engine.py
├── ai/                  # ML components
│   ├── processor.py
│   └── enrichment.py
├── integrations/        # External connectors
│   └── connectors.py
├── workers/             # Celery tasks
│   └── tasks.py
├── utils/               # Utilities
│   └── audit.py
└── celery_app.py        # Celery config
```

### Running Tests
```bash
# Unit tests
pytest tests/unit/

# Integration tests
pytest tests/integration/

# With coverage
pytest --cov=app tests/
```

### Code Quality
```bash
# Lint
ruff check .

# Format
black .

# Type checking
mypy app/
```

### Database Migrations
```bash
# Create migration
alembic revision --autogenerate -m "Description"

# Apply
alembic upgrade head

# Downgrade
alembic downgrade -1
```

## Production Deployment

### Using Docker
```bash
# Build production image
docker build -t cybersec-alert-automation:latest .

# Deploy with docker-compose
docker-compose -f docker-compose.prod.yml up -d
```

### Kubernetes
See `k8s/` directory for manifests:
- Deployment
- Service
- Ingress
- ConfigMap
- Secret
- PersistentVolumeClaims

### Environment Variables
Set these in production:
```bash
DEBUG=False
SECRET_KEY=<strong-random-key>
DATABASE_URL=postgresql://...
ENABLE_EMAIL_NOTIFICATIONS=True
SMTP_SERVER=smtp.example.com
```

### Monitoring
- Prometheus metrics at `/metrics`
- Health checks at `/health`
- Structured JSON logging
- Celery Flower for task monitoring

## Alert Mitigation Workflow

1. **Ingestion**
   - Alert received via API
   - Basic validation
   - Database record created

2. **AI Analysis**
   - Feature extraction
   - ML classification
   - Risk scoring
   - False positive detection

3. **Enrichment**
   - GeoIP lookup
   - Threat intel
   - Asset context

4. **Rule Processing**
   - Evaluate all enabled rules (by priority)
   - Apply actions: filter, escalate, suppress, mitigate, notify
   - Record rule executions

5. **Mitigation**
   - If AI score > threshold, schedule automated actions
   - Execute playbook steps
   - Track status

6. **Resolution**
   - Analyst can acknowledge, investigate, resolve
   - Full audit trail maintained

## Security Considerations

### API Security
- JWT authentication required for most endpoints
- Rate limiting recommended for alert ingestion
- Validate and sanitize all inputs
- Use HTTPS in production

### Data Protection
- Sensitive data encrypted at rest
- Audit logs immutable
- Regular backups
- Access controls

### Network Security
- Deploy in isolated network segment
- Firewall rules restrict access
- VPN for admin access

## Troubleshooting

### Common Issues

**Database connection failed**
```bash
# Check DATABASE_URL
echo $DATABASE_URL
# Verify PostgreSQL is running
docker-compose ps postgres
```

**Redis connection refused**
```bash
# Check Redis
docker-compose logs redis
```

**AI model not loading**
```bash
# Train a new model
python -c "from app.ai.processor import ai_processor; ai_processor.train_from_historical(db)"
```

### Logs
```bash
# Application logs
docker-compose logs -f app

# Worker logs
docker-compose logs -f worker

# Database logs
docker-compose logs -f postgres
```

## Contributing

Contributions welcome! Please:
1. Fork the repository
2. Create feature branch
3. Add tests
4. Ensure linting passes
5. Submit PR

## License

MIT License - See LICENSE file for details.

## Support

For issues, bugs, or feature requests, please open an issue on GitHub.

---

**Note**: This system is designed for enterprise security environments. Always test in a controlled environment before production deployment. Regular tuning of rules and AI model retraining is recommended for optimal performance.
