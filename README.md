# StageFinder - Job Application Management System

[![PHP Version](https://img.shields.io/badge/PHP-8.2%2B-blue)](https://www.php.net/)
[![Symfony](https://img.shields.io/badge/Symfony-6.3-black)](https://symfony.com/)
[![License](https://img.shields.io/badge/license-proprietary-red)](LICENSE)

StageFinder is a comprehensive job application management system that helps users find companies, generate personalized cover letters, and manage their job applications with automated email sending capabilities.

## Features

- 🔍 **Company Search**: Search for French companies using INSEE SIRENE API
- 📄 **CV Management**: Secure encrypted storage for CVs
- 📝 **Template System**: Create and manage cover letter templates with placeholders
- 🎯 **PDF Generation**: Generate professional PDF cover letters
- 📧 **Email Automation**: Send applications via user's own SMTP configuration
- ⭐ **Favorites & Tags**: Organize companies with favorites and tags
- 🔐 **Security**: Encrypted file storage, secure authentication
- 🗂️ **Application Tracking**: Track all sent applications with status
- 🔒 **GDPR Compliant**: Full data deletion support

## Technology Stack

- **Backend**: Symfony 6.3, PHP 8.2+
- **Database**: MySQL 8.0 / MariaDB
- **Cache**: Redis
- **PDF Generation**: Dompdf
- **Email**: Symfony Mailer with SMTP
- **Queue**: Symfony Messenger
- **Encryption**: Libsodium
- **Containerization**: Docker & Docker Compose

## Prerequisites

- Docker & Docker Compose
- Git
- (Optional) Composer if running locally without Docker
- (Optional) SIRENE API token from INSEE

## Quick Start with Docker

### 1. Clone the repository

```bash
git clone <repository-url>
cd stagefinder
```

### 2. Configure environment

Create `.env.local` file:

```bash
cp .env .env.local
```

Edit `.env.local` with your configuration:

```env
APP_ENV=dev
APP_SECRET=change_this_to_a_random_32_char_string

# Database
DATABASE_URL=mysql://symfony:symfony@db:3306/stagefinder

# Redis
REDIS_URL=redis://redis:6379

# Encryption key (32 bytes base64 encoded)
# Generate with: echo -n "your-32-byte-secret-key-here!" | base64
ENCRYPTION_KEY=base64:YW5leGFtcGxlMzJieXRlc2VjcmV0a2V5MTIzNDU2Nzg=

# File storage
CLOUD_STORAGE_PATH=/var/www/uploads

# SIRENE API (optional - for company sync)
SIRENE_TOKEN=your_insee_api_token_here

# Messenger
MESSENGER_TRANSPORT_DSN=doctrine://default
```

### 3. Start Docker containers

```bash
docker-compose up -d
```

This will start:
- MySQL database (port 3306)
- Redis (port 6379)
- PHP-FPM application
- Nginx web server (port 8080)
- Worker for async tasks

### 4. Install dependencies & setup database

```bash
# Enter the app container
docker-compose exec app bash

# Install dependencies
composer install

# Create database schema
php bin/console doctrine:migrations:migrate --no-interaction

# Create admin user (optional)
php bin/console app:create-admin admin@example.com password123 "Admin Name"

# Seed sample companies for Chartres (optional)
php bin/console app:seed-companies
```

### 5. Access the application

- **Web interface**: http://localhost:8080
- **API**: http://localhost:8080/api

## Manual Installation (Without Docker)

### Requirements

- PHP 8.2+ with extensions: pdo_mysql, zip, intl, gd, mbstring, sodium
- MySQL 8.0+
- Composer
- Redis (optional but recommended)

### Installation Steps

```bash
# Install dependencies
composer install

# Configure database in .env.local
DATABASE_URL=mysql://user:password@localhost:3306/stagefinder

# Create database
php bin/console doctrine:database:create

# Run migrations
php bin/console doctrine:migrations:migrate

# Start Symfony server
symfony server:start

# In another terminal, start worker
php bin/console messenger:consume async -vv
```

## Usage Guide

### 1. Register and Login

```bash
# Create account via API
curl -X POST http://localhost:8080/api/register \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"secure123","fullName":"John Doe"}'

# Login
curl -X POST http://localhost:8080/api/login \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"secure123"}'
```

### 2. Search Companies

```bash
# Search in Chartres with 25km radius
curl "http://localhost:8080/api/companies?zones=chartres,28&rad=25&page=1"
```

### 3. Upload CV

```bash
curl -X POST http://localhost:8080/api/cv/upload \
  -F "file=@/path/to/your/cv.pdf"
```

### 4. Create Template

```bash
curl -X POST http://localhost:8080/api/templates \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My Cover Letter",
    "templateText": "Madame, Monsieur,\n\nJe vous écris pour postuler chez {{COMPANY_NAME}}..."
  }'
```

### 5. Generate Cover Letter

```bash
curl -X POST http://localhost:8080/api/generate-cover \
  -H "Content-Type: application/json" \
  -d '{
    "company_id": 1,
    "template_id": 1,
    "cv_id": 1
  }'
```

### 6. Configure SMTP

```bash
curl -X POST http://localhost:8080/api/smtp/connect \
  -H "Content-Type: application/json" \
  -d '{
    "host": "smtp.gmail.com",
    "port": 587,
    "username": "your-email@gmail.com",
    "password": "your-app-password",
    "tls": true
  }'
```

### 7. Send Application

```bash
curl -X POST http://localhost:8080/api/send \
  -H "Content-Type: application/json" \
  -d '{
    "company_id": 1,
    "cover_pdf_id": 1,
    "cv_id": 1,
    "smtp_config_id": 1,
    "subject": "Candidature spontanée",
    "body_override": "Veuillez trouver ci-joint..."
  }'
```

## Available Placeholders

Use these in your templates:

- `{{COMPANY_NAME}}` - Company name
- `{{COMPANY_ADDRESS}}` - Full company address
- `{{COMPANY_CITY}}` - Company city
- `{{COMPANY_POSTAL_CODE}}` - Postal code
- `{{USER_FULLNAME}}` - Your full name
- `{{USER_EMAIL}}` - Your email
- `{{CURRENT_DATE}}` - Current date
- `{{POSITION}}` - Position (if specified)

You can also use defaults:
```
{{COMPANY_EMAIL | default('recrutement@company.com')}}
```

## Console Commands

### Sync Companies from SIRENE

```bash
php bin/console app:sync-companies --city=Chartres --radius=25
```

### Create Admin User

```bash
php bin/console app:create-admin admin@example.com password123 "Admin User"
```

### Seed Sample Data

```bash
php bin/console app:seed-companies
```

### Consume Messages (Worker)

```bash
php bin/console messenger:consume async -vv
```

### Clean Old Files

```bash
php bin/console app:prune-old-files --days=90
```

## Testing

### Run Unit Tests

```bash
./vendor/bin/phpunit
```

### Run Specific Test

```bash
./vendor/bin/phpunit tests/Service/TemplateRendererTest.php
```

### Test with Coverage

```bash
./vendor/bin/phpunit --coverage-html coverage
```

## API Documentation

Complete API documentation is available in `/docs/API.md`

### Main Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | /api/register | Create new account |
| POST | /api/login | Login |
| GET | /api/me | Get current user |
| GET | /api/companies | Search companies |
| GET | /api/companies/{id} | Get company details |
| POST | /api/favorites | Add favorite |
| GET | /api/favorites | List favorites |
| POST | /api/templates | Create template |
| GET | /api/templates | List templates |
| POST | /api/cv/upload | Upload CV |
| POST | /api/generate-cover | Generate cover letter |
| POST | /api/smtp/connect | Configure SMTP |
| POST | /api/send | Send application |
| GET | /api/applications | List applications |
| POST | /api/account/delete | Delete account (GDPR) |

## Security Features

- ✅ Password hashing with Argon2id
- ✅ File encryption with Libsodium
- ✅ CSRF protection
- ✅ Rate limiting on sensitive endpoints
- ✅ Input validation and sanitization
- ✅ SQL injection prevention (Doctrine ORM)
- ✅ XSS prevention in templates
- ✅ Secure SMTP credential storage

## GDPR Compliance

- Users can delete their account and all associated data
- Encrypted file storage
- Data minimization
- Consent management
- Right to access (export data)
- Right to erasure

## Troubleshooting

### Database Connection Error

```bash
# Check if MySQL is running
docker-compose ps

# Check logs
docker-compose logs db
```

### Worker Not Processing Jobs

```bash
# Check worker logs
docker-compose logs worker

# Restart worker
docker-compose restart worker
```

### Permission Errors

```bash
# Fix permissions
docker-compose exec app chown -R www-data:www-data /var/www/uploads
docker-compose exec app chmod -R 755 /var/www/html
```

### Clear Cache

```bash
php bin/console cache:clear
```

## Production Deployment

### Environment Configuration

```env
APP_ENV=prod
APP_DEBUG=0
APP_SECRET=<strong-random-secret>
DATABASE_URL=mysql://user:pass@prod-db:3306/stagefinder
```

### Optimization

```bash
# Install production dependencies
composer install --no-dev --optimize-autoloader

# Build assets
php bin/console assets:install --env=prod

# Clear and warm up cache
php bin/console cache:clear --env=prod
php bin/console cache:warmup --env=prod
```

### Security Checklist

- [ ] Change APP_SECRET to random value
- [ ] Change ENCRYPTION_KEY
- [ ] Use strong database passwords
- [ ] Enable HTTPS
- [ ] Set secure headers in Nginx
- [ ] Configure firewall rules
- [ ] Set up monitoring and logging
- [ ] Regular backups
- [ ] Keep dependencies updated

## Monitoring

### Health Check Endpoint

```bash
curl http://localhost:8080/api/health
```

### Logs

```bash
# Application logs
tail -f var/log/prod.log

# Docker logs
docker-compose logs -f app
docker-compose logs -f worker
```

## Contributing

1. Fork the repository
2. Create feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open Pull Request

## License

Proprietary - All rights reserved

## Support

For issues and questions:
- Create an issue on GitHub
- Contact: support@stagefinder.example

## Roadmap

- [ ] LinkedIn integration
- [ ] Multi-language support
- [ ] Advanced analytics
- [ ] Mobile app
- [ ] AI-powered cover letter suggestions
- [ ] Integration with more company databases
- [ ] Calendar integration for interview scheduling

---

Made with ❤️ for job seekers
