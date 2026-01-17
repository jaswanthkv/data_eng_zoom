# NY Taxi Data Engineering Pipeline

A complete data engineering solution for ingesting and managing NYC yellow taxi trip data using Docker, PostgreSQL, and pgAdmin.

## 📋 Project Overview

This project demonstrates a modern data engineering pipeline that:
- **Ingests** NYC yellow taxi data from Parquet files (2021 data)
- **Stores** data in PostgreSQL with configurable table names
- **Visualizes** data through pgAdmin web interface
- **Scales** with support for multiple months/years of data
- **Automates** with Docker containerization

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│                   Data Pipeline                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  NYC Taxi Data (Parquet)                                │
│          │                                              │
│          ▼                                              │
│  ┌──────────────────────┐                               │
│  │  Ingest Service      │                               │
│  │  (Docker Container)  │                               │
│  │  - Download data     │                               │
│  │  - Transform chunks  │                               │
│  │  - Load to DB        │                               │
│  └──────────┬───────────┘                               │
│             │                                           │
│             ▼                                           │
│  ┌──────────────────────┐      ┌──────────────────────┐│
│  │   PostgreSQL 18      │──────│   pgAdmin 7.8        ││
│  │   Port: 5432         │      │   Port: 8085         ││
│  │                      │      │                      ││
│  │ Tables:              │      │ Query/Manage DB      ││
│  │ - yellow_taxi_*      │      │ - Browse tables      ││
│  │ - jaswanth_taxi      │      │ - Execute SQL        ││
│  │ - custom_*           │      │ - Export data        ││
│  └──────────────────────┘      └──────────────────────┘│
│                                                         │
└─────────────────────────────────────────────────────────┘
```

## 🚀 Quick Start

### Prerequisites
- Docker & Docker Compose installed
- 4GB+ RAM available
- Internet connection for data download

### 1. Start Services

```bash
cd pipeline
docker-compose up -d
```

This starts:
- ✅ PostgreSQL database on `localhost:5432`
- ✅ pgAdmin UI on `http://localhost:8085`

### 2. Access pgAdmin

- **URL**: http://localhost:8085
- **Email**: admin@admin.com
- **Password**: root

### 3. Ingest Data

```bash
docker-compose --profile manual run --rm ingest \
  --pg-user=root \
  --pg-pass=root \
  --pg-host=pgdatabase \
  --pg-port=5432 \
  --pg-db=ny_taxi \
  --target-table=yellow_taxi_2021_02 \
  --year=2021 \
  --month=2 \
  --chunksize=100000
```

### 4. Query Data in pgAdmin

1. Navigate to: Servers → ny_taxi → Databases → ny_taxi → Schemas → public → Tables
2. Right-click a table → View/Edit Data → First 100 rows
3. Or use SQL Editor to write custom queries

## 📊 Usage

### Ingest Multiple Months

```bash
# February 2021
docker-compose --profile manual run --rm ingest \
  --pg-user=root --pg-pass=root --pg-host=pgdatabase --pg-port=5432 --pg-db=ny_taxi \
  --target-table=yellow_taxi_2021_02 --year=2021 --month=2 --chunksize=100000

# March 2021
docker-compose --profile manual run --rm ingest \
  --pg-user=root --pg-pass=root --pg-host=pgdatabase --pg-port=5432 --pg-db=ny_taxi \
  --target-table=yellow_taxi_2021_03 --year=2021 --month=3 --chunksize=100000

# April 2021
docker-compose --profile manual run --rm ingest \
  --pg-user=root --pg-pass=root --pg-host=pgdatabase --pg-port=5432 --pg-db=ny_taxi \
  --target-table=yellow_taxi_2021_04 --year=2021 --month=4 --chunksize=100000
```

### Query Data via psql

```bash
# Connect to database
docker exec -it pipeline-pgdatabase-1 psql -U root -d ny_taxi

# List tables
\dt

# Count records
SELECT COUNT(*) FROM yellow_taxi_2021_02;

# Sample query
SELECT 
  tpep_pickup_datetime, 
  trip_distance, 
  total_amount
FROM yellow_taxi_2021_02 
LIMIT 10;
```

### Export Data

In pgAdmin:
1. Right-click table → Backup
2. Select format (SQL, CSV, etc.)
3. Save to local machine

## 📁 Project Structure

```
data_eng_zoom/
├── README.md                 # This file
├── pipeline/
│   ├── docker-compose.yaml   # Docker Compose configuration
│   ├── Dockerfile            # Ingest service image
│   ├── ingest_data.py        # Data ingestion script
│   ├── main.py              # Pipeline orchestration
│   ├── pipeline.py          # Pipeline utilities
│   ├── pyproject.toml       # Python dependencies
│   ├── servers.json         # pgAdmin server config
│   └── pgadmin_servers.conf # pgAdmin servers config file
├── ny_taxi_postgres_data/   # Database volume (local)
└── notebook.ipynb           # Jupyter notebook for analysis
```

## 🔧 Configuration

### Ingest Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `--pg-user` | root | PostgreSQL username |
| `--pg-pass` | root | PostgreSQL password |
| `--pg-host` | pgdatabase | PostgreSQL host |
| `--pg-port` | 5432 | PostgreSQL port |
| `--pg-db` | ny_taxi | Database name |
| `--target-table` | yellow_taxi_trips | Output table name |
| `--year` | 2021 | Year of data |
| `--month` | 1 | Month of data (1-12) |
| `--chunksize` | 100000 | Records per batch |

### Database Connection

From your local machine:
```
Host: localhost
Port: 5432
Username: root
Password: root
Database: ny_taxi
```

## 🐛 Troubleshooting

### Port Already in Use

```bash
# Find process using port 5432
lsof -i :5432

# Kill process or change port in docker-compose.yaml
```

### pgAdmin Blank Screen

```bash
# Restart pgAdmin
docker restart pipeline-pgadmin-1

# Clear browser cache and hard refresh (Ctrl+Shift+R)
```

### Connection Refused During Ingest

Ensure you use `--pg-host=pgdatabase` (not localhost) when running through docker-compose.

### Check Container Logs

```bash
# Database logs
docker logs pipeline-pgdatabase-1

# pgAdmin logs
docker logs pipeline-pgadmin-1

# Ingest logs
docker logs pipeline-ingest-1
```

## 📈 Performance Tips

- **Chunk Size**: Increase `--chunksize` to 500000 for faster ingestion on powerful machines
- **Indexes**: Create indexes on frequently queried columns after ingestion
- **Compression**: Use pgAdmin's backup with compression for large exports

## 🔐 Security Notes

⚠️ **Development Only**: This setup uses default credentials and is not production-ready.

For production:
- Use strong passwords
- Enable SSL/TLS
- Implement authentication
- Use secrets management
- Run behind firewall

## 📝 Data Schema

Tables created follow this structure:

```sql
CREATE TABLE yellow_taxi_2021_02 (
  vendor_id BIGINT,
  tpep_pickup_datetime TIMESTAMP,
  tpep_dropoff_datetime TIMESTAMP,
  passenger_count BIGINT,
  trip_distance FLOAT,
  rate_code_id BIGINT,
  store_and_fwd_flag TEXT,
  pickup_location_id BIGINT,
  dropoff_location_id BIGINT,
  payment_type BIGINT,
  fare_amount FLOAT,
  extra FLOAT,
  mta_tax FLOAT,
  tip_amount FLOAT,
  tolls_amount FLOAT,
  total_amount FLOAT
);
```

## 🛑 Shutdown

```bash
# Stop containers (keep data)
docker-compose down

# Stop containers and remove volumes (delete data)
docker-compose down -v
```

## 📚 References

- [NYC Taxi Data](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [pgAdmin Documentation](https://www.pgadmin.org/docs/)
- [Docker Compose](https://docs.docker.com/compose/)

## 👤 Author

Created as part of data engineering learning journey.

## 📄 License

MIT License - feel free to use this project for learning purposes.