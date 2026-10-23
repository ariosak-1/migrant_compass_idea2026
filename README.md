
# Migrant Compass

Migrant Compass is a prototype for finding verified support services, viewing reported hazards, and comparing geographic information along selected routes.

This README describes the proposed implementation. The folder layout, endpoints, and commands become operational as the corresponding files are implemented.

## 1. Technology Stack

| Component | Technology | Responsibility |
|---|---|---|
| Frontend | React + TypeScript + Vite | Build the interface based on the Figma design |
| Map renderer | MapLibre GL JS | Display basemap tiles, service markers, hazard layers, and route lines |
| Backend | Python + FastAPI | Receive app requests, validate inputs, query data, and return responses |
| Database | PostgreSQL + PostGIS | Store records and perform geographic searches |
| Database access | SQLAlchemy + psycopg + GeoAlchemy2 | Connect Python to PostgreSQL and work with geographic fields |
| Database migrations | Alembic | Track and apply database structure changes |
| External requests | HTTPX | Request data from external providers |
| Routing, later | OSMnx + NetworkX | Load road/path networks and calculate candidate routes |
| Local environment | Docker Compose | Run the database and backend consistently |

PostgreSQL stores the application's data. PostGIS adds support for geographic points, lines, areas, distances, and intersections.

MapLibre renders the map. A separately configured tile/style provider supplies the basemap.

## 2. Initial Scope

Start with:

- One city or short corridor.
- One travel mode.
- A small directory of verified public services.
- Weather information.
- Source and freshness information for displayed records.

Add incident layers, route comparisons, and multilingual assistance after the basic data pipeline works.

Reported hazards and geographic calculations support informed decisions; they do not establish that a route is safe.

## 3. Data Sources

| Source | Data to use | Application use |
|---|---|---|
| OpenStreetMap | Roads, paths, bridges, and mapped facilities | Geographic reference and routing network |
| Open-Meteo | Temperature, apparent temperature, precipitation, wind, and forecast times | Weather information for selected locations and travel times |
| ACLED | Reported events, event types, dates, locations, and precision information | Context about relevant reported incidents |
| OCHA HDX | Selected country-specific boundaries, infrastructure, and facility datasets | Geographic reference layers and additional facilities |
| Local NGOs and municipal providers | Public service locations, hours, eligibility, languages, and contacts | Locally verified service directory |
| UNHCR HELP | Country-specific public guidance and support information | Additional service and information references |

Select exact datasets after choosing the pilot area.

For every source, document:

- Dataset or resource identifier.
- Geographic coverage.
- Source URL.
- Format.
- License and attribution requirements.
- Expected update schedule.
- Actual source timestamps.
- Known limitations.
- Who maintains the integration.

ACLED requires authentication. Available update frequency depends on access level and geography.

HDX is a collection of datasets and resources. Each selected resource may need different parsing and validation.

## 4. Repository Structure

Files marked [later] can wait until the basic map works.

```text
migrant-compass/

  README.md
  # Main project documentation.
  # Explain purpose, architecture, setup, commands, and build order.
  # Link the Figma design and methodology documentation.
  # Update this when the actual implementation changes.

  compose.yaml
  # Defines local Docker services.
  #
  # db:
  # - PostgreSQL with PostGIS installed.
  # - Persistent named volume for database records.
  # - Database health check.
  #
  # api:
  # - Builds the backend Docker image.
  # - Receives backend environment variables.
  # - Waits for a healthy database.
  # - Exposes backend port 8000.
  # - Mounts development fixtures read-only at /data/fixtures.
  # - Can mount backend source code for development reloads.
  #
  # Schedule import commands separately from the API server.
  # They can use the same backend image.

  .env.example
  # Backend configuration names with placeholder values.
  # Copy to an untracked .env file and set local values.
  #
  # Suggested settings:
  # POSTGRES_DB
  # POSTGRES_USER
  # POSTGRES_PASSWORD
  # DATABASE_URL
  # ALLOWED_ORIGINS
  # ACLED authentication settings
  # MODEL_API_KEY                    [later]
  #
  # compose.yaml must explicitly pass required settings to services.

  .gitignore
  # Exclude actual environment files and secrets.
  # Exclude Python environments, caches, and node_modules.
  # Exclude downloaded datasets, generated graphs, and database dumps.
  # Keep small synthetic fixtures in version control.

  web/

    package.json
    # Frontend dependencies and commands.
    # Dependencies include React and MapLibre GL JS.
    #
    # Define scripts:
    # dev        -> Start the development server.
    # build      -> Produce the deployable frontend.
    # typecheck  -> Check TypeScript.
    # lint       -> Check code quality.

    package-lock.json
    # Records exact frontend dependency versions.
    # Commit this so developers install the same dependencies.

    .env.example
    # Public frontend configuration only.
    #
    # Example:
    # VITE_API_BASE_URL=http://localhost:8000
    #
    # Also document the selected map style/tile configuration.
    # VITE_ variables are exposed in the browser bundle.
    # Keep database passwords and private provider keys in the backend.

    index.html
    # Browser entry document.
    # Contains the element where React mounts.

    vite.config.ts
    # Frontend build and development-server configuration.
    # Configure the React plugin and development port.

    tsconfig.json
    # TypeScript configuration.
    # Enable strict checking to catch incorrect data usage.

    src/

      main.tsx
      # Starts React and mounts App into index.html.
      # Imports global styles and MapLibre's stylesheet.

      App.tsx
      # Top-level application layout.
      # Initially displays MapPage.
      # Add navigation when more screens are introduced.

      pages/

        MapPage.tsx
        # Coordinates the map screen.
        # Stores the selected area, category, and service.
        # Calls functions from api/client.ts.
        # Passes returned features to MapView.
        # Passes the selected service to ServiceCard.
        #
        # Implement:
        # - Loading state.
        # - Request error state.
        # - Empty results state.
        # - Stale-data notices.
        # - Cancellation of obsolete requests.
        # - Debouncing when map movement triggers requests.

      components/

        MapView.tsx
        # Creates and manages the MapLibre map.
        # Loads the configured basemap style.
        # Displays service locations from GeoJSON.
        # Reports selected markers to MapPage.
        # Cleans up the map instance when unmounted.
        #
        # Later:
        # - Incident layers.
        # - Hazard areas.
        # - Candidate route lines.
        # - Layer visibility controls.

        ServiceCard.tsx
        # Displays one selected service.
        #
        # Show:
        # - Name and category.
        # - Opening information.
        # - Eligibility.
        # - Supported languages.
        # - Public contact information.
        # - Source.
        # - Verification status and date.
        #
        # Represent unavailable fields explicitly.

        HazardLegend.tsx
        # Explains symbols and categories.
        # Shows the relevant time period and source freshness.
        # Distinguishes unknown information from low reported exposure.

      api/

        client.ts
        # Central location for requests to your FastAPI backend.
        # Reads VITE_API_BASE_URL.
        # Builds query strings using URLSearchParams.
        # Checks response.ok before processing successful responses.
        # Handles errors and cancellation.
        #
        # Suggested functions:
        # getNearbyServices(...)
        # getService(...)
        # getHazards(...)
        # getWeather(...)
        # assessRoute(...)               [later]
        # askAssistant(...)              [later]

      styles/

        global.css
        # App-wide typography, colors, spacing, and layout.
        # Translate Figma design tokens into CSS variables.
        # Include mobile layouts and visible keyboard focus states.

  backend/

    Dockerfile
    # Builds the Python application image.
    # Installs dependencies and copies application files.
    # Sets the working directory.
    # Starts the API through Uvicorn.
    # Supports running import commands from the same image.

    requirements.txt
    # Compatible, pinned runtime dependencies.
    #
    # Likely packages:
    # fastapi
    # uvicorn
    # sqlalchemy
    # psycopg
    # geoalchemy2
    # alembic
    # httpx
    # pydantic-settings
    #
    # Add osmnx and networkx when routing is implemented.

    requirements-dev.txt
    # Development dependencies such as pytest and a Python linter.
    # Install these in the development/test environment.

    alembic.ini
    # Database migration configuration.
    # Obtain the connection URL from backend settings.

    migrations/
    # Version-controlled database change scripts.
    #
    # Initial migrations:
    # - Enable PostGIS.
    # - Create services and data_sources.
    # - Add geographic indexes.
    # - Add constraints preventing duplicate source records.
    #
    # Later migrations:
    # - Add incidents.
    # - Add weather_forecasts.
    # - Add ingestion_runs.
    #
    # Add a migration whenever the database schema changes.

    app/

      __init__.py
      # Marks app as a Python package.
      # Add __init__.py inside other Python package directories too.

      main.py
      # Creates the FastAPI application.
      # Registers endpoint modules.
      # Configures allowed frontend origins through CORS.
      # Sets up database lifecycle handling.
      # Provides development API documentation.

      config.py
      # Reads and validates backend environment settings.
      #
      # Include:
      # - Database URL.
      # - Allowed frontend origins.
      # - Provider authentication settings.
      # - Request timeouts.
      # - Supported pilot area.
      # - Limits on search radius and returned records.
      #
      # Fail clearly when required settings are missing.

      schemas.py
      # Pydantic request and response definitions.
      # Defines required fields, optional fields, enums, and limits.
      #
      # Validate:
      # - Latitude and longitude ranges.
      # - Search radius and result limits.
      # - Supported categories and travel modes.
      # - Time ranges.
      # - Timezone-aware departure times.
      #
      # Define exactly what the frontend receives.
      # Keep internal credentials and ingestion details out of responses.

      api/

        health.py
        # GET /health/live
        # Confirms the server process is running.
        #
        # GET /health/ready
        # Checks required dependencies such as the database.

        services.py
        # GET /api/services/nearby
        #
        # Parameters:
        # lat, lon, radius_m, category, limit.
        #
        # Responsibilities:
        # - Validate the request.
        # - Call services/nearby_services.py.
        # - Return public service features and distance information.
        #
        # GET /api/services/{service_id}
        # Returns one public service record.
        # Returns 404 when the record does not exist.
        #
        # Register /nearby before /{service_id}.

        hazards.py
        # GET /api/hazards
        #
        # Parameters:
        # bbox, start_time, end_time, hazard_type, limit.
        #
        # Responsibilities:
        # - Validate bounds and requested time range.
        # - Query stored records for that area and period.
        # - Return geometry, category, source, and timestamps.
        # - Preserve available date/location precision.
        # - Indicate result truncation or provide pagination.

        weather.py
        # GET /api/weather
        #
        # Parameters:
        # lat, lon, and an optional requested forecast window.
        #
        # Responsibilities:
        # - Retrieve matching stored forecast information.
        # - Return measurements with units.
        # - Include valid times, fetched time, and freshness.
        # - Represent unavailable forecasts explicitly.

        route_assessments.py
        # [later] POST /api/route-assessments
        #
        # Request body:
        # origin, destination, travel_mode, departure_at.
        #
        # Responsibilities:
        # - Validate the supported area and travel mode.
        # - Call routing and exposure services.
        # - Return candidate route geometry.
        # - Include distance, estimated duration, explanations,
        #   source dates, and data-coverage gaps.
        # - Return a defined result when no supported route exists.

        assistant.py
        # [later] POST /api/chat
        #
        # Request body:
        # message, language, and relevant context references.
        #
        # Responsibilities:
        # - Retrieve relevant public records and computed results.
        # - Pass selected context to the model.
        # - Return an explanation with supporting references.
        # - Apply request size and usage limits.

      db/

        session.py
        # Configures SQLAlchemy database connections.
        # Supplies sessions to requests and import jobs.
        # Commits successful writes.
        # Rolls back failed transactions.
        # Closes sessions after use.

        models.py
        # Defines database tables and relationships.
        #
        # Start with:
        # - services
        # - data_sources
        #
        # Add:
        # - incidents
        # - weather_forecasts
        # - ingestion_runs
        #
        # Include:
        # - Primary keys.
        # - Stable source identifiers.
        # - Geographic columns.
        # - Geographic indexes.
        # - Uniqueness constraints.
        # - Relevant source and application timestamps.

      sources/

        open_meteo.py
        # Makes HTTP requests to the Open-Meteo Forecast API.
        # Sends coordinates, variables, and time settings.
        # Uses a timeout and checks HTTP status.
        # Returns provider data for validation and normalization.

        acled.py
        # Authenticates using configured ACLED credentials.
        # Manages OAuth tokens and refresh.
        # Requests events for the selected area/time window.
        # Handles pagination.
        # Retrieves information needed for corrections and deletions.

        hdx.py
        # Retrieves metadata for selected HDX datasets.
        # Identifies the resource selected for this project.
        # Downloads that resource or queries supported tabular data.
        # Records resource ID, source date, license, and download time.
        # Checks the expected file format and size.

        osm.py
        # [later] Obtains an area-specific network using OSMnx.
        # Chooses the correct travel mode.
        # Records extent, retrieval date, and provenance.
        # Saves a reproducible network snapshot.

      ingestion.py
      # Converts provider responses into your internal representation.
      #
      # Responsibilities:
      # - Validate required fields.
      # - Validate coordinates.
      # - Preserve or convert units explicitly.
      # - Preserve event dates and forecast valid times.
      # - Record download time separately.
      # - Preserve the source's precision.
      # - Identify existing records using stable keys.
      # - Update records without creating duplicates.
      # - Reject or quarantine malformed records.
      #
      # Add bounded retry handling for suitable transient failures.
      # Avoid treating missing measurements as zero.

      services/

        nearby_services.py
        # Performs geographic service searches in PostGIS.
        # Filters by category and relevant verification status.
        # Calculates geographic distance.
        # Orders results and applies limits.
        # Returns records to api/services.py.

        exposure.py
        # [later] Compares route segments with relevant hazard records.
        # Considers location precision, event age, and forecast time.
        # Returns separate indicators and explanations.
        # Records the assessment-method version.
        # Represents missing evidence explicitly.

        routing.py
        # [later] Loads the prepared road/path graph.
        # Finds candidate routes for the requested travel mode.
        # Applies documented constraints and route weights.
        # Handles disconnected or unsupported routes explicitly.

      jobs/

        import_services.py
        # Runnable command for importing curated services.
        # Accepts a CSV or GeoJSON file path.
        # Validates, normalizes, and writes records.
        # Reports inserted, updated, and rejected record counts.
        #
        # Intended command:
        # python -m app.jobs.import_services --file <path>

        refresh_weather.py
        # Runnable command for updating configured forecast locations.
        # Calls sources/open_meteo.py.
        # Uses ingestion.py to validate and normalize.
        # Saves successful results transactionally.
        # Retains existing valid data if the refresh fails.

        refresh_incidents.py
        # Runnable command for synchronizing incident records.
        # Processes additions, corrections, and source deletions.
        # Records the last successful synchronization checkpoint.

        build_network.py
        # [later] Downloads, validates, and saves a network snapshot.
        # Runs separately from requests made by map users.

    tests/

      test_imports.py
      # Checks repeat imports, corrections, deletions,
      # malformed records, and failed provider requests.

      test_geographic_queries.py
      # Checks coordinate order, distance units, and area filters.

      test_api.py
      # Checks input validation, response fields,
      # missing records, result limits, and unavailable data.
      #
      # Database tests must use a separate test database.

  data/

    fixtures/

      services.geojson
      # Small synthetic service dataset.

      weather.json
      # Synthetic normal, missing, and stale weather examples.

      incidents.json
      # Synthetic incident examples with varied precision.
      #
      # These fixtures are for development and checks.
      # They are not real operational guidance.

  docs/

    architecture.md
    # Explains running components and their connections.
    # Documents local ports and deployment dependencies.
    # Explains scheduled imports versus user requests.

    data_sources.md
    # Source inventory and integration notes.
    # Include IDs, URLs, coverage, formats, permissions,
    # timestamps, update schedules, limitations, and ownership.

    risk_method.md
    # Documents indicators, constraints, and any route weights.
    # Explains assumptions and missing-data handling.
    # Records validation results and reviewers.
    # Version this whenever the assessment method changes.
```

## 5. What the Backend Layers Do

| Layer | Responsibility | Example |
|---|---|---|
| `api/` | Receive and validate app requests | “Find nearby clinics” |
| `services/` | Perform queries and calculations | Search PostGIS and calculate distance |
| `sources/` | Communicate with external providers | Request weather from Open-Meteo |
| `ingestion.py` | Validate and normalize imported data | Convert provider fields into internal records |
| `jobs/` | Run complete import workflows | Fetch, validate, save, and record the outcome |
| `db/` | Define tables and manage database access | Open a transaction and save records |

The frontend communicates with FastAPI. Backend components access PostgreSQL and private provider credentials.

## 6. Database Tables

### services

Suggested fields:

- `id`
- `source_id`
- `source_record_id`
- `name`
- `category`
- `location`
- `opening_hours`
- `eligibility`
- `languages`
- `public_contact`
- `verification_status`
- `last_verified_at`
- `source_updated_at`
- `fetched_at`

Use a geographic point for `location`.

Keep verification separate from import time: downloading a service record today does not establish that its opening hours were verified today.

### data_sources

Suggested fields:

- `id`
- `provider`
- `dataset_id`
- `resource_id`
- `source_url`
- `license`
- `expected_update_schedule`

### incidents

Suggested fields:

- `id`
- `source_id`
- `source_event_id`
- `event_date`
- `event_type`
- `location`
- `location_precision`
- `date_precision`
- `source_updated_at`
- `fetched_at`

Preserve the precision supplied by the source. An event recorded only to a date should not be assigned an invented exact time.

### weather_forecasts

Suggested fields:

- `id`
- `source_id`
- `location`
- `valid_at`
- `forecast_run_at`, when supplied
- `temperature`
- `apparent_temperature`
- `precipitation`
- `wind_speed`
- `units`
- `fetched_at`

Define stable keys so repeated imports of the same forecast records do not create duplicates.

### ingestion_runs

Suggested fields:

- `id`
- `source_id`
- `job_name`
- `started_at`
- `finished_at`
- `status`
- `records_inserted`
- `records_updated`
- `records_rejected`
- `error_summary`
- `successful_checkpoint`

Update the successful synchronization checkpoint only after the import completes successfully.

## 7. Geographic Conventions

Use consistent geographic conventions across the application:

- GeoJSON positions use `[longitude, latitude]`.
- Bounding boxes use `west,south,east,north`.
- Use WGS84 coordinates for GeoJSON exchange.
- Include units with measurements.
- Use timezone-aware timestamps.
- Distinguish event time, forecast valid time, source update time, and download time.

For PostGIS distance queries:

- A suitable `geography` column allows distances in metres.
- Geometry distance units depend on the coordinate system.
- Geographic proximity is different from walking distance.
- Walking distance requires calculations along a valid network.

## 8. API Requests: Frontend to Backend

These endpoints are proposed application contracts.

`GET` retrieves information. `POST` submits a request body for processing.

| Request | Purpose | Response |
|---|---|---|
| `GET /health/live` | Check whether the API process is running | Basic server status |
| `GET /health/ready` | Check required dependencies | Readiness status |
| `GET /api/services/nearby` | Find nearby services | GeoJSON features with relevant service and distance information |
| `GET /api/services/{service_id}` | Retrieve one service | Public service details |
| `GET /api/hazards` | Retrieve hazards for an area and period | Features, sources, timestamps, and coverage information |
| `GET /api/weather` | Retrieve available forecasts | Values, units, valid times, and freshness |
| `POST /api/route-assessments` | Assess candidate routes | Geometry, distance, duration estimates, explanations, and gaps |
| `POST /api/chat` | Explain retrieved information | Answer with supporting references |

### Example service request

The following coordinates are examples, not a selected pilot area.

```http
GET /api/services/nearby?lat=52.52&lon=13.405&radius_m=5000&category=clinic&limit=100
```

The backend should:

1. Validate coordinates, category, radius, and limit.
2. Query services within the requested geographic distance.
3. Calculate and order distances.
4. Return public service fields.
5. Include verification information.
6. Indicate whether more results are available.

### Example hazard request

```http
GET /api/hazards?bbox=13.3,52.4,13.5,52.6&limit=100
```

Add `start_time`, `end_time`, and `hazard_type` parameters as their contracts are implemented.

### Example frontend request function

Place the request logic in `web/src/api/client.ts`.

```typescript
export async function getNearbyServices(
  latitude: number,
  longitude: number,
  category: string,
  signal?: AbortSignal
) {
  // Public configuration pointing to your FastAPI server.
  const baseUrl = import.meta.env.VITE_API_BASE_URL;

  const url = new URL("/api/services/nearby", baseUrl);

  // URLSearchParams handles query-string encoding.
  url.search = new URLSearchParams({
    lat: String(latitude),
    lon: String(longitude),
    radius_m: "5000",
    category,
    limit: "100",
  }).toString();

  // signal allows the caller to cancel an obsolete request.
  const response = await fetch(url, { signal });

  if (!response.ok) {
    throw new Error(`Service request failed: ${response.status}`);
  }

  // Define a matching response type and validate data as needed.
  return response.json();
}
```

`MapPage.tsx` calls this function and manages loading, error, and cancellation state. `MapView.tsx` receives the resulting features for display.

## 9. API Requests: Backend to Data Providers

Provider requests belong in `backend/app/sources/`.

### Open-Meteo

The core of `sources/open_meteo.py` could be:

```python
import httpx


def fetch_weather(latitude: float, longitude: float) -> dict:
    # Request selected hourly variables for a specific location.
    params = {
        "latitude": latitude,
        "longitude": longitude,
        "hourly": (
            "temperature_2m,"
            "apparent_temperature,"
            "precipitation,"
            "wind_speed_10m"
        ),
        "forecast_days": 2,
        "timezone": "GMT",
        "timeformat": "unixtime",
    }

    # Use a timeout so the job cannot wait indefinitely.
    with httpx.Client(timeout=15.0) as client:
        response = client.get(
            "https://api.open-meteo.com/v1/forecast",
            params=params,
        )

        # Raise an error for unsuccessful HTTP responses.
        # The job runner should record the failure.
        response.raise_for_status()

        # Return provider JSON for validation and normalization.
        return response.json()
```

After receiving the response:

- Validate the structure.
- Read returned units.
- Match values to their returned timestamps.
- Handle missing values explicitly.
- Save the forecast valid time separately from download time.

The first hourly entry is not necessarily the current hour.

### ACLED

Implement the following in `sources/acled.py`:

1. Authenticate using configured ACLED credentials.
2. Obtain and refresh OAuth tokens.
3. Request the chosen geographic and temporal subset.
4. Handle pagination.
5. Preserve source event identifiers.
6. Process corrections and deletions.
7. Respect the available access level and update schedule.

Filter event types deliberately. ACLED contains violent and nonviolent events.

### HDX

Implement the following in `sources/hdx.py`:

1. Inspect the selected dataset using metadata endpoints such as `package_show`.
2. Identify the specific resource selected for the application.
3. Check its format, permissions, and relevant dates.
4. Download the resource or query a supported tabular endpoint.
5. Parse it using resource-specific rules.
6. Normalize and validate records before saving.

Select resources deliberately rather than automatically importing every file in a dataset.

### OpenStreetMap

When routing is introduced:

1. Select the pilot area and travel mode.
2. Retrieve the relevant network with OSMnx.
3. Validate connectivity and local suitability.
4. Save a reproducible snapshot.
5. Use NetworkX for candidate-route calculations.
6. Record the snapshot date and source attribution.

## 10. Import Job Workflow

Example workflow for `refresh_weather.py`:

```text
# 1. Load configured forecast locations.
# 2. Create an ingestion_runs record.
# 3. Call sources/open_meteo.py.
# 4. Validate and normalize using ingestion.py.
# 5. Write accepted records in a database transaction.
# 6. Record inserted, updated, and rejected counts.
# 7. Mark the run successful.
# 8. On failure, record the error and retain existing valid data.
```

Imports should be repeatable.

Use stable provider identifiers or documented keys to update existing records without creating duplicates.

Run jobs manually while developing. Add a separate scheduler after imports work reliably. Opening the map should not require every external provider to respond.

## 11. Local Development Setup

These commands describe the intended workflow after the scaffold, dependencies, Compose services, migrations, and import CLI are implemented.

### Configure environment files

From the repository root:

```bash
cp .env.example .env
cp web/.env.example web/.env
```

Edit both files.

Inside Docker, the backend database hostname should match the Compose database service name, such as `db`.

The frontend API base URL should point to the browser-accessible backend, such as:

```text
VITE_API_BASE_URL=http://localhost:8000
```

Configure the backend to allow the frontend origin:

```text
http://localhost:5173
```

### Start the database and apply migrations

```bash
docker compose build api
docker compose up -d --wait db
docker compose run --rm api alembic upgrade head
```

The database service must have a working health check.

### Import synthetic service fixtures

This command assumes the importer implements `--file` and Compose mounts the fixture directory at `/data/fixtures`.

```bash
docker compose run --rm api python -m app.jobs.import_services --file /data/fixtures/services.geojson
```

### Start the backend

```bash
docker compose up api
```

### Start the frontend

In another terminal:

```bash
cd web
npm ci
npm run dev
```

Expected development addresses:

- Frontend: `http://localhost:5173`
- Backend: `http://localhost:8000`
- FastAPI documentation: `http://localhost:8000/docs`

## 12. Development Order

### Milestone 1: Environment

- Create the repository and frontend/backend scaffolds.
- Configure Docker Compose.
- Start PostgreSQL with PostGIS.
- Implement health endpoints.
- Confirm the frontend can reach the backend.

### Milestone 2: Database-backed service map

- Create the first migration.
- Add synthetic service fixtures.
- Implement the service importer.
- Implement nearby-service queries.
- Add the service API endpoints.
- Display returned locations on the map.
- Open a service card when a marker is selected.

The first complete feature should connect:

`models.py → migration → import_services.py → nearby_services.py → api/services.py → web/api/client.ts → MapView.tsx`

### Milestone 3: Real data import

- Replace synthetic services with a reviewed source.
- Add source references and verification information.
- Test repeated imports and malformed records.
- Record ingestion outcomes.

### Milestone 4: Weather

- Implement the Open-Meteo client.
- Normalize and store forecasts.
- Add the weather endpoint.
- Display units, valid times, and freshness.
- Test unavailable and stale forecasts.

### Milestone 5: Incident and geographic layers

- Integrate selected ACLED and HDX resources.
- Preserve event categories and precision.
- Add geographic and time filtering.
- Display sources and coverage limitations.

### Milestone 6: Route assessment

- Prepare the network for the selected area and travel mode.
- Establish baseline routing.
- Add documented exposure calculations.
- Validate assumptions with GIS and local reviewers.
- Return explanations and data gaps.

### Milestone 7: Multilingual assistance

- Retrieve relevant records and computed results.
- Pass selected context to the model.
- Return supporting references.
- Review responses in the supported languages.

## 13. Checks and Collaboration

For each feature:

1. Create a short-lived Git branch.
2. Implement the smallest complete behavior.
3. Run relevant checks.
4. Review the change.
5. Merge it.
6. Update documentation when behavior changes.

Prioritize checks for:

- Reversed latitude and longitude.
- Incorrect distance units.
- Duplicate imports.
- Corrected and deleted source records.
- Stale or unavailable forecasts.
- Missing values incorrectly treated as zero.
- Unsupported areas and travel modes.
- Disconnected routing networks.
- API validation and result limits.

Use a separate test database for checks that create, update, or delete database records.

## 14. Pilot Deployment

Prepare a staging environment before a supervised pilot.

Configure:

- A deployed frontend.
- A deployed API.
- PostgreSQL with PostGIS and persistent storage.
- Separate scheduled import jobs.
- HTTPS.
- Backend secrets.
- Restricted database access.
- Database backups.
- Monitoring for failed imports and stale sources.

Apply migrations as a controlled deployment step.

Minimize collection and retention of precise user locations. Keep public service information separate from any user-specific data.

## 15. Official Documentation

- FastAPI application structure: https://fastapi.tiangolo.com/tutorial/bigger-applications/
- FastAPI CORS: https://fastapi.tiangolo.com/tutorial/cors/
- PostgreSQL: https://www.postgresql.org/about/
- PostGIS setup: https://postgis.net/documentation/getting_started/
- PostGIS distance queries: https://postgis.net/docs/ST_DWithin.html
- Alembic: https://alembic.sqlalchemy.org/en/latest/tutorial.html
- MapLibre GL JS: https://maplibre.org/maplibre-gl-js/docs/
- Vite environment variables: https://vite.dev/guide/env-and-mode
- HTTPX: https://www.python-httpx.org/quickstart/
- Open-Meteo: https://open-meteo.com/en/docs
- ACLED API: https://acleddata.com/api-documentation/getting-started
- HDX API: https://docs.humdata.org/build/overview/hdx-api-overview
- OSMnx: https://osmnx.readthedocs.io/en/stable/getting-started.html
- OpenStreetMap regional downloads: https://download.geofabrik.de/
- OpenStreetMap attribution and license: https://www.openstreetmap.org/copyright
- GeoJSON specification: https://www.rfc-editor.org/rfc/rfc7946
- Docker Compose: https://docs.docker.com/compose/
