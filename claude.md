# Blazer

## Project Overview

Blazer is a Ruby on Rails engine that enables users to explore their data with SQL. It provides an intuitive interface for creating charts and dashboards, and sharing them with teams. The project is battle-tested at Instacart and supports multiple database backends.

**Key Features:**
- Multiple data sources support (PostgreSQL, MySQL, Redshift, BigQuery, Athena, and 20+ more)
- SQL query interface with variables
- Automatic chart generation from query results
- Dashboard creation and management
- Data checks and email alerts
- Query auditing and caching
- Cohort analysis
- Anomaly detection and forecasting
- CSV upload support

## Technology Stack

**Backend:**
- Ruby 3.1+
- Rails 7+ (Rails Engine architecture)
- ActiveRecord 6.1+

**Frontend:**
- Vue.js 3 (production build)
- jQuery
- Chart.js with Chartkick
- Bootstrap 3
- ACE Editor for SQL editing
- Selectize.js for dropdowns
- Moment.js for date handling
- Mapbox GL JS for maps

**Additional Libraries:**
- safely_block for error handling
- Various database adapter gems (pg, mysql2, google-cloud-bigquery, aws-sdk-athena, etc.)

## Project Structure

```
blazer/
├── app/
│   ├── assets/          # JavaScript, CSS, fonts, images
│   ├── controllers/     # Rails controllers (queries, dashboards, checks, uploads)
│   ├── models/          # ActiveRecord models (query, dashboard, check, audit, upload)
│   ├── views/           # ERB templates
│   └── helpers/         # View helpers
├── lib/
│   ├── blazer/
│   │   ├── adapters/    # Database adapter implementations (20+ adapters)
│   │   ├── data_source.rb
│   │   ├── engine.rb    # Rails engine definition
│   │   ├── run_statement.rb
│   │   ├── result.rb
│   │   ├── result_cache.rb
│   │   ├── check_mailer.rb
│   │   ├── slack_notifier.rb
│   │   ├── forecasters.rb
│   │   └── anomaly_detectors.rb
│   └── generators/      # Rails generators for installation
├── config/
│   └── routes.rb        # Engine routes
├── test/                # Test suite
└── blazer.gemspec       # Gem specification
```

## Core Components

### Models

- **Blazer::Query** (app/models/blazer/query.rb) - SQL queries with metadata
- **Blazer::Dashboard** (app/models/blazer/dashboard.rb) - Collection of queries
- **Blazer::Check** (app/models/blazer/check.rb) - Data quality checks with alerts
- **Blazer::Audit** (app/models/blazer/audit.rb) - Query execution audit log
- **Blazer::Upload** (app/models/blazer/upload.rb) - CSV upload tracking

### Controllers

- **Blazer::QueriesController** (app/controllers/blazer/queries_controller.rb) - Query CRUD and execution
- **Blazer::DashboardsController** (app/controllers/blazer/dashboards_controller.rb) - Dashboard management
- **Blazer::ChecksController** (app/controllers/blazer/checks_controller.rb) - Check management
- **Blazer::UploadsController** (app/controllers/blazer/uploads_controller.rb) - CSV uploads

### Database Adapters

All adapters inherit from `Blazer::Adapters::BaseAdapter` and implement:
- `run_statement(statement, options)` - Execute SQL/query
- `tables` - List available tables
- `schema` - Get table schema information
- `preview_statement` - Preview query results

**Key Adapters:**
- SQL Adapter (lib/blazer/adapters/sql_adapter.rb) - Base for SQL databases
- PostgreSQL, MySQL, SQLite - Via sql_adapter
- BigQuery (lib/blazer/adapters/bigquery_adapter.rb)
- Athena (lib/blazer/adapters/athena_adapter.rb)
- Snowflake (lib/blazer/adapters/snowflake_adapter.rb)
- Elasticsearch/OpenSearch
- And 15+ more specialized adapters

### Key Library Files

- **lib/blazer/data_source.rb** - Manages connections to data sources
- **lib/blazer/run_statement.rb** - Executes queries with timeout and security
- **lib/blazer/result.rb** - Query result processing and chart detection
- **lib/blazer/result_cache.rb** - Query result caching
- **lib/blazer.rb** - Main module with configuration

## Architecture

Blazer is a Rails Engine that mounts into a host Rails application:

1. Host app includes `gem "blazer"` in Gemfile
2. Run `rails generate blazer:install` to create migrations and config
3. Mount engine in routes: `mount Blazer::Engine, at: "blazer"`
4. Configure data sources in `config/blazer.yml`

The engine provides complete MVC stack but integrates with host app's:
- Authentication system (via before_action callbacks)
- Asset pipeline or Propshaft
- Job queue (optional for async queries)
- Cache store (for query caching)

## Configuration

Main config file: `config/blazer.yml` in host Rails app

Key settings:
- `data_sources` - Database connections
- `timeout` - Query timeout (default 15 seconds)
- `cache` - Result caching configuration
- `async` - Run queries asynchronously
- `smart_variables` - Dropdown options for query variables
- `linked_columns` - URL patterns for column values
- `smart_columns` - Join data without SQL joins
- `check_schedules` - When to run checks
- `anomaly_checks` - Anomaly detection method
- `forecasting` - Forecasting method

## Development Guidelines

### Code Style

- Follow Ruby community style guide
- Use 2-space indentation
- Keep methods focused and small
- Use descriptive variable names

### Adding a New Adapter

1. Create new file in `lib/blazer/adapters/`
2. Inherit from `Blazer::Adapters::BaseAdapter`
3. Implement required methods: `run_statement`, `tables`, `schema`
4. Register adapter: `Blazer.register_adapter "name", AdapterClass`
5. Add tests in `test/adapters/`

### Testing

- Test suite in `test/` directory
- Adapter tests in `test/adapters/`
- Feature tests for queries, charts, dashboards, checks
- Uses internal Rails app in `test/internal/` for testing

### Security Considerations

- Always use read-only database users
- Queries run in transactions and rollback (when possible)
- All queries are audited
- Supports authentication via host app
- SQL injection protection via parameterized queries
- Content Security Policy overrides available

## Common Tasks

### Running Tests
```bash
bundle exec rake test
```

### Testing a Specific Adapter
```bash
bundle exec rake test TEST=test/adapters/postgresql_test.rb
```

### Running Checks
```bash
rake blazer:run_checks SCHEDULE="5 minutes"
rake blazer:send_failing_checks
```

### Archiving Old Queries
```bash
rake blazer:archive_queries
```

## Important Paths

- Main entry point: `lib/blazer.rb`
- Engine definition: `lib/blazer/engine.rb`
- Query execution: `lib/blazer/run_statement.rb`
- Result processing: `lib/blazer/result.rb`
- Chart detection logic: `lib/blazer/result.rb` (chart detection based on column types)
- Query interface: `app/assets/javascripts/blazer/queries.js`
- SQL editor: ACE editor integration in views

## Chart Types

Blazer automatically detects and generates charts based on column types:

- **Line Chart**: timestamp + numeric(s) OR timestamp + string + numeric
- **Column Chart**: string + numeric(s) OR string + string + numeric
- **Scatter Chart**: numeric + numeric
- **Pie Chart**: string + numeric (with column named "pie")
- **Map**: latitude/longitude columns OR geojson column

## External Dependencies

Optional gems for extended functionality:
- `prophet-rb` - Forecasting and anomaly detection
- `trend` - Alternative forecasting
- `anomaly_detection` - Alternative anomaly detection
- Database-specific gems (pg, mysql2, google-cloud-bigquery, etc.)

## Resources

- Demo: https://blazer.dokkuapp.com
- Repository: https://github.com/ankane/blazer
- Docker image: https://github.com/ankane/blazer-docker
- Dev app: https://github.com/ankane/blazer-dev
