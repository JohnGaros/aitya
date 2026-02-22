# Open/Public Datasets and APIs for Energy Sector Decision Support Systems

Research compiled 2026-02-22. European context with emphasis on Greece; global sources included where relevant.

---

## Table of Contents

1. [Electricity Markets](#1-electricity-markets)
2. [Natural Gas](#2-natural-gas)
3. [Renewables](#3-renewables)
4. [Weather & Climate](#4-weather--climate)
5. [Carbon & Emissions](#5-carbon--emissions)
6. [Grid & Infrastructure](#6-grid--infrastructure)
7. [Regulatory & Policy](#7-regulatory--policy)
8. [Energy Consumption & Efficiency](#8-energy-consumption--efficiency)
9. [Commodity Prices](#9-commodity-prices)
10. [Greek-Specific Energy Data](#10-greek-specific-energy-data)
11. [Aggregator Platforms & Meta-Sources](#11-aggregator-platforms--meta-sources)
12. [Kaggle & Academic Datasets](#12-kaggle--academic-datasets)
13. [Open Source Energy Models with Data](#13-open-source-energy-models-with-data)
14. [Python Packages Summary](#14-python-packages-summary)
15. [Pricing Data Summary](#pricing-data-summary)

---

## 1. Electricity Markets

### 1.1 ENTSO-E Transparency Platform

**What it is:** The central data platform operated by European Transmission System Operators, mandated by EU Regulation 543/2013. The most comprehensive source of European electricity market data.

**Data fields and granularity:**
- **Day-ahead prices** (data item 12.1.D) -- hourly EUR/MWh per bidding zone (including Greece `GR`)
- **Actual total load** (6.1.A) -- 15-minute/hourly resolution per control area
- **Day-ahead load forecast** (6.1.B) -- hourly per bidding zone
- **Generation per production type** (16.1.B&C) -- hourly, broken down by fuel type (lignite, gas, hydro, wind onshore/offshore, solar, biomass, nuclear, etc.). Note: only units >= 100 MW are mandated to report
- **Cross-border physical flows** (12.1.G) -- hourly flows between bidding zones
- **Scheduled commercial exchanges** (12.1.D) -- day-ahead and intraday
- **Transmission capacity** (NTC values, 11.1.A) -- available transfer capacity between bidding zones
- **Installed generation capacity per type** (14.1.A) -- yearly per bidding zone
- **Balancing market data** -- activated balancing energy, prices, procured capacity
- **Outage data** -- planned and forced outages for generation units and transmission lines
- **Generation forecasts** -- wind and solar day-ahead/intraday forecasts

**How to collect in bulk:**

1. **RESTful API:**
   - Base URL: `https://web-api.tp.entsoe.eu/api`
   - Authentication: `securityToken` parameter (free registration required)
   - Example for day-ahead prices: `GET /api?securityToken=TOKEN&documentType=A44&in_Domain=10YGR-HTSO-----Y&out_Domain=10YGR-HTSO-----Y&periodStart=202501010000&periodEnd=202501312300`
   - Rate limit: 400 requests/user/minute (IP + token based); exceeding triggers 10-min ban
   - Returns XML by default

2. **File Library (bulk CSV downloads):**
   - Token endpoint: `https://keycloak.tp.entsoe.eu/realms/tp/protocol/openid-connect/token`
   - File library endpoint: `https://fms.tp.entsoe.eu`
   - Tab-delimited CSV files (UTF-8), organized into yearly, monthly, or daily files

3. **Python package `entsoe-py`:**
   ```bash
   pip install entsoe-py
   ```
   ```python
   from entsoe import EntsoePandasClient
   import pandas as pd

   client = EntsoePandasClient(api_key='YOUR_TOKEN')
   start = pd.Timestamp('2025-01-01', tz='Europe/Athens')
   end = pd.Timestamp('2025-02-01', tz='Europe/Athens')

   # Day-ahead prices for Greece
   prices = client.query_day_ahead_prices('GR', start=start, end=end)

   # Generation by fuel type
   generation = client.query_generation('GR', start=start, end=end, psr_type=None)

   # Cross-border flows
   flows = client.query_crossborder_flows('GR', 'BG', start=start, end=end)

   # Total load
   load = client.query_load('GR', start=start, end=end)
   ```

4. **Alternative Python package `entsoe-apy`:**
   ```bash
   pip install entsoe-apy
   ```

**Format:** XML (API), CSV (File Library), Pandas DataFrames (Python client)

**Update frequency:** Near real-time for operational data; historical data available from January 2015 onward.

**Licensing:** Free registration required. Email `transparency@entsoe.eu` with subject "RESTful API access" to get your security token. Data is publicly available under ENTSO-E data policy.

**Decision support use cases:**
- Day-ahead and intraday electricity price forecasting
- Generation mix optimization and portfolio management
- Cross-border arbitrage analysis
- Load forecasting and demand-side management
- Renewable generation curtailment analysis
- Transmission congestion analysis
- Market coupling and price convergence studies

**Key links:**
- Platform: https://transparency.entsoe.eu/
- API guide: https://transparency.entsoe.eu/content/static_content/Static%20content/web%20api/Guide.html
- Postman collection: https://documenter.getpostman.com/view/7009892/2s93JtP3F6
- entsoe-py GitHub: https://github.com/EnergieID/entsoe-py

---

### 1.2 EPEX SPOT (European Power Exchange)

**What it is:** Power exchange operating day-ahead and intraday markets for 15+ European countries (Austria, Belgium, Denmark, Germany, Finland, France, Luxembourg, Netherlands, Norway, Poland, Sweden, UK, Switzerland). Greece is coupled via HEnEx.

**Data fields:**
- Day-ahead auction prices (hourly, moving to 15-minute resolution from Q3/Q4 2025)
- Intraday continuous trading prices and volumes
- Aggregated buy/sell curves
- Block order results

**How to collect:**
- **Official access:** Subscription-based via FTP/webshare (Excel/CSV files). Paid service -- prices vary by data usage type.
- **No free public API.** EPEX data is commercial.
- **Workaround via ENTSO-E:** Day-ahead prices from EPEX-coupled markets are mirrored on ENTSO-E Transparency Platform (free).
- **Third-party:** Energy-Charts API (Fraunhofer ISE) provides EPEX price data for free (see Section 11).

**Licensing:** Commercial subscription required for direct access. ENTSO-E mirror is free.

**Key links:**
- Market data services: https://www.epexspot.com/en/marketdataservices

---

### 1.3 Nord Pool

**What it is:** Power exchange serving Nordic, Baltic, and expanding European markets. Day-ahead (Elspot) and intraday (Elbas) markets.

**Data fields:**
- Day-ahead prices per bidding zone (hourly, transitioning to 15-minute)
- Buy/sell volumes
- System price
- Capacities and flows
- Intraday trading data

**How to collect:**
- **Public data portal:** https://data.nordpoolgroup.com/ (day-ahead prices viewable/downloadable)
- **Market Data API:** Available to Nord Pool customers (trading API is free for customers; others must become customers). Product sheet at https://www.nordpoolgroup.com/globalassets/download-center/power-data-services/market-data-api.pdf
- **Python library:**
  ```bash
  pip install nordpool
  ```
  Unofficial library for fetching Elspot prices: https://github.com/kipe/nordpool
- **Kaggle mirror:** "Nord Pool Energy Market Data" dataset available at https://www.kaggle.com/datasets/pythonafroz/nord-pool-energy-market-data

**Licensing:** Free viewing on data portal; API requires customer status. Kaggle mirror freely available.

---

## 2. Natural Gas

### 2.1 ENTSOG Transparency Platform

**What it is:** The central platform for European natural gas transmission data, operated by European Network of Transmission System Operators for Gas.

**Data fields and granularity:**
- **Physical flows** at all network points (daily, some hourly)
- **Firm and interruptible capacity** (technical, booked, available)
- **Nominations and renominations**
- **Allocations**
- **Quality indicators** (gross calorific value, Wobbe index)
- **Data at multiple network point types:** transmission, storage, LNG, production, and interconnection points
- Covers entire EU gas network including Greek interconnections (TAP, IGI Poseidon, Bulgarian border)

**How to collect:**

1. **REST API (public, no registration):**
   - Base URL: `https://transparency.entsog.eu/api/v1/`
   - Operational data: `https://transparency.entsog.eu/api/v1/operationaldata?periodType=day&from=2025-01-01&to=2025-01-31&indicator=Physical%20Flow`
   - Operator point directions: `https://transparency.entsog.eu/api/v1/operatorpointdirections.xml`
   - Max query execution time: 60 seconds
   - Best practice: request monthly ranges; avoid large indicator sets in single calls
   - Returns JSON or XML
   - API manual (v3.0, October 2025): https://transparency.entsog.eu/api/archiveDirectories/8/api-manual/ENTSOG_TP_API_UserManual_v3.0.pdf

2. **Python package `entsog-py`:**
   ```bash
   pip install entsog-py
   ```
   ```python
   from entsog import EntsogPandasClient
   import pandas as pd

   client = EntsogPandasClient()
   start = pd.Timestamp('2025-01-01', tz='Europe/Athens')
   end = pd.Timestamp('2025-02-01', tz='Europe/Athens')

   # Query operational data
   data = client.query_operational_data(
       start=start, end=end,
       country_code='GR',
       indicator=['Physical Flow', 'GCV']
   )
   ```

**Format:** JSON, XML (API); Pandas DataFrames (Python client)

**Update frequency:** Daily updates; some data near real-time.

**Licensing:** Fully public, no registration or API key required.

**Decision support use cases:**
- Gas flow monitoring and supply security analysis
- Gas-electricity coupling models (gas as marginal fuel for power generation)
- LNG terminal utilization analysis
- Cross-border gas flow optimization
- Storage injection/withdrawal strategy

**Key links:**
- Platform: https://transparency.entsog.eu/
- Gas flow dashboard: https://gasdashboard.entsog.eu/
- entsog-py GitHub: https://github.com/nhcb/entsog-py

---

### 2.2 GIE AGSI+ and ALSI (Gas Storage & LNG Terminals)

**What it is:** Gas Infrastructure Europe's transparency platforms for underground gas storage (AGSI+) and large-scale LNG terminals (ALSI). 100% coverage of EU27 storage and LNG facilities.

**Data fields:**
- **AGSI+ (gas storage):** filling level (TWh, %), injection/withdrawal flow (GWh/day), working gas volume, working capacity, injection/withdrawal capacity. Daily resolution.
- **ALSI (LNG terminals):** LNG inventory, send-out flow, storage capacity, regasification capacity. Daily resolution.
- Since April 2025: contracted and available capacities also published daily.
- Data at facility, operator, and country level.

**How to collect:**

1. **REST API:**
   - Requires free API key (register at https://agsi.gie.eu/ or https://alsi.gie.eu/)
   - Set "access to" = "Both ALSI and AGSI+" during registration
   - API documentation: https://www.gie.eu/transparency-platform/GIE_API_documentation_v007.pdf

2. **Python package `gie-py`:**
   ```bash
   pip install gie-py
   ```
   ```python
   from gie import GiePandasClient

   client = GiePandasClient(api_key='YOUR_API_KEY')

   # Gas storage data for Greece
   df = client.query_gas_country('GR', start='2024-01-01', end='2025-01-01')

   # LNG terminal data (e.g., Revithoussa)
   df_lng = client.query_lng_country('GR', start='2024-01-01', end='2025-01-01')
   ```

**Format:** JSON (API), Pandas DataFrames (Python)

**Update frequency:** Daily

**Licensing:** Free with registration (API key required).

**Decision support use cases:**
- Gas storage strategy optimization (injection/withdrawal timing)
- Security of supply monitoring
- LNG terminal utilization and scheduling
- Winter preparedness assessment
- Gas price forecasting (storage levels are a key driver)

**Key links:**
- AGSI+ platform: https://agsi.gie.eu/
- ALSI platform: https://alsi.gie.eu/
- gie-py GitHub: https://github.com/fboerman/gie-py

---

### 2.3 TTF and European Gas Hub Prices

**What it is:** Title Transfer Facility (TTF) in the Netherlands is Europe's benchmark gas hub. Other hubs include THE (Germany), PEG (France), PSV (Italy).

**Data fields:**
- Day-ahead prices (EUR/MWh)
- Futures prices (monthly, quarterly, seasonal, calendar year)
- Historical spot and forward curves

**How to collect:**
- **ICE (official exchange):** https://www.ice.com/products/27996665/Dutch-TTF-Natural-Gas-Futures/data -- free delayed data; real-time requires subscription
- **EEX (European Energy Exchange):** https://www.eex.com/en/market-data/market-data-hub/natural-gas/indices -- EEX Neutral Gas Price TTF; downloadable history of last 60 days free
- **Oil Price API (free tier):**
  ```
  GET https://api.oilpriceapi.com/v1/prices/latest?by_code=DUTCH_TTF_EUR
  ```
  Free tier available, no credit card required.
- **Commodities-API (freemium):**
  ```
  GET https://commodities-api.com/api/latest?access_key=YOUR_KEY&base=EUR&symbols=TFMI
  ```
  Historical since April 2023.
- **Trading Economics:** https://tradingeconomics.com/commodity/eu-natural-gas (visual/download, limited free tier)

**Licensing:** Varies by source. Official exchange data is delayed or paid; third-party APIs offer free tiers with rate limits.

**Decision support use cases:**
- Gas procurement strategy
- Power price forecasting (gas is the marginal fuel in most European markets)
- Gas-electricity spread analysis (spark spread, clean spark spread)

---

## 3. Renewables

### 3.1 PVGIS (Photovoltaic Geographical Information System)

**What it is:** Free tool from the European Commission Joint Research Centre providing solar radiation data and PV system performance estimates for any location in the world (except poles). Uses ECMWF ERA-5 and satellite-based CMSAF datasets.

**Data fields:**
- **Hourly solar radiation time series:** Global Horizontal Irradiance (GHI), Direct Normal Irradiance (DNI), Diffuse Horizontal Irradiance (DHI), plane-of-array irradiance
- **PV power output estimates:** for grid-connected and standalone systems
- **Typical Meteorological Year (TMY):** 8760-hour synthetic year from historical data
- **Monthly/yearly averages** of solar radiation and PV output
- **Temperature, wind speed** at location
- Coverage: global, resolution ~30 km (ERA-5 based), satellite data higher resolution for Europe/Africa

**How to collect:**

1. **Non-interactive API:**
   - Base URL: `https://re.jrc.ec.europa.eu/api/v5_3/`
   - Hourly data: `https://re.jrc.ec.europa.eu/api/v5_3/seriescalc?lat=37.98&lon=23.73&startyear=2015&endyear=2020&outputformat=json`
   - TMY data: `https://re.jrc.ec.europa.eu/api/v5_3/tmy?lat=37.98&lon=23.73&outputformat=json`
   - PV estimate: `https://re.jrc.ec.europa.eu/api/v5_3/PVcalc?lat=37.98&lon=23.73&peakpower=1&loss=14&outputformat=json`
   - Rate limit: 30 calls/second per IP
   - Output: JSON or CSV

2. **Python via `pvlib`:**
   ```bash
   pip install pvlib
   ```
   ```python
   from pvlib.iotools import get_pvgis_hourly, get_pvgis_tmy

   # Hourly irradiance data for Athens
   data, _, _ = get_pvgis_hourly(latitude=37.98, longitude=23.73,
                                  start=2015, end=2020,
                                  components=True)

   # TMY data
   tmy_data, _, _, _ = get_pvgis_tmy(latitude=37.98, longitude=23.73)
   ```

3. **Bulk download (gridded datasets):**
   - https://joint-research-centre.ec.europa.eu/photovoltaic-geographical-information-system-pvgis/pvgis-data-download_en

**Format:** JSON, CSV, EPW (TMY), NetCDF (bulk gridded data)

**Update frequency:** Database updated periodically (current version PVGIS 5.3 based on ERA-5 data through recent years).

**Licensing:** Free and open access. EU Joint Research Centre.

**Decision support use cases:**
- Solar PV project siting and capacity estimation
- Solar generation forecasting
- Building energy performance analysis
- Grid integration planning for solar
- Bankability assessment for solar project financing

**Key links:**
- Web tool: https://joint-research-centre.ec.europa.eu/photovoltaic-geographical-information-system-pvgis/pvgis-tools_en
- API docs: https://joint-research-centre.ec.europa.eu/photovoltaic-geographical-information-system-pvgis/getting-started-pvgis/api-non-interactive-service_en

---

### 3.2 NASA POWER

**What it is:** Prediction of Worldwide Energy Resources. Provides solar and meteorological data from NASA research, suitable for renewable energy and building energy efficiency applications.

**Data fields:**
- Solar irradiance: GHI, DNI, DHI (surface and top-of-atmosphere)
- Temperature (2m, 10m)
- Wind speed at configurable elevations (10m--300m)
- Relative humidity, precipitation
- Surface albedo
- Cloud cover
- Resolution: 0.5 x 0.5 degree (~50 km), daily/hourly/monthly
- Coverage: global, from 1981 onward (daily), 2001 onward (hourly)

**How to collect:**

1. **REST API (no registration required):**
   - Daily: `https://power.larc.nasa.gov/api/temporal/daily/point?start=20200101&end=20201231&latitude=37.98&longitude=23.73&community=RE&parameters=ALLSKY_SFC_SW_DWN,T2M,WS10M&format=JSON`
   - Hourly: `https://power.larc.nasa.gov/api/temporal/hourly/point?start=20200101&end=20200131&latitude=37.98&longitude=23.73&community=RE&parameters=ALLSKY_SFC_SW_DWN,WS50M&format=JSON`
   - Monthly: `https://power.larc.nasa.gov/api/temporal/monthly/point?start=2010&end=2020&latitude=37.98&longitude=23.73&community=RE&parameters=ALLSKY_SFC_SW_DWN&format=JSON`
   - Max 20 parameters per daily request, 15 per hourly request
   - Returns JSON or CSV

2. **Data Access Viewer (interactive):** https://power.larc.nasa.gov/data-access-viewer/

**Format:** JSON, CSV, ASCII, NetCDF, ICASA

**Update frequency:** Data updated with ~5-day latency.

**Licensing:** Completely free, no registration required. NASA open data policy.

**Decision support use cases:**
- Renewable energy resource assessment (especially for locations without ground stations)
- Building energy efficiency modeling
- Agricultural energy modeling
- Backup/validation for satellite-derived solar data

**Key links:**
- API overview: https://power.larc.nasa.gov/docs/services/api/
- Parameter docs: https://power.larc.nasa.gov/docs/tutorials/parameters/

---

### 3.3 IRENA Statistics

**What it is:** International Renewable Energy Agency's comprehensive global renewable energy database.

**Data fields:**
- Installed renewable capacity by technology and country (MW) -- from 2000 onward
- Renewable power generation (GWh) by technology and country
- Renewable energy balances
- Off-grid electricity statistics
- Published annually (capacity in March, generation in July)

**How to collect:**
- **Online query tool:** https://pxweb.irena.org/pxweb/en/IRENASTAT/
- **Bulk downloads:** https://www.irena.org/Data/Downloads/IRENASTAT (Excel format)
- **PDF reports:** https://www.irena.org/Publications/2025/Jul/Renewable-energy-statistics-2025

**Format:** Excel (XLSX), interactive web queries (PxWeb)

**Update frequency:** Annual

**Licensing:** Free and open.

**Decision support use cases:**
- Country-level renewable capacity planning
- Benchmarking renewable deployment across countries
- Long-term energy transition modeling

---

## 4. Weather & Climate

### 4.1 Open-Meteo

**What it is:** Free, open-source weather API providing current, forecast, and historical weather data. No API key required for non-commercial use. Uses data from national weather services and ECMWF.

**Data fields:**
- **Forecast (up to 16 days):** temperature, humidity, precipitation, wind speed/direction/gusts, pressure, cloud cover, solar radiation (GHI, DNI, DHI), UV index. Hourly resolution.
- **Historical (1940--present via ERA5):** same variables, hourly, ~10 km resolution
- **Satellite Radiation API:** dedicated solar radiation endpoint from satellite datasets
- **Climate projections:** CMIP6 climate model outputs

**How to collect:**

1. **REST API (no key required for non-commercial use):**
   - Forecast: `https://api.open-meteo.com/v1/forecast?latitude=37.98&longitude=23.73&hourly=temperature_2m,windspeed_10m,shortwave_radiation,direct_normal_irradiance`
   - Historical: `https://archive-api.open-meteo.com/v1/archive?latitude=37.98&longitude=23.73&start_date=2023-01-01&end_date=2023-12-31&hourly=temperature_2m,windspeed_10m,shortwave_radiation`
   - Satellite radiation: `https://api.open-meteo.com/v1/satellite-radiation?latitude=37.98&longitude=23.73&hourly=shortwave_radiation,direct_normal_irradiance`
   - Returns JSON (or FlatBuffers via Python client)

2. **Python package:**
   ```bash
   pip install openmeteo-requests requests-cache retry-requests
   ```
   ```python
   import openmeteo_requests
   import requests_cache
   from retry_requests import retry

   cache_session = requests_cache.CachedSession('.cache', expire_after=3600)
   retry_session = retry(cache_session, retries=5, backoff_factor=0.2)
   om = openmeteo_requests.Client(session=retry_session)

   params = {
       "latitude": 37.98, "longitude": 23.73,
       "hourly": ["temperature_2m", "windspeed_10m", "shortwave_radiation"],
       "start_date": "2023-01-01", "end_date": "2023-12-31"
   }
   responses = om.weather_api("https://archive-api.open-meteo.com/v1/archive", params=params)
   ```

3. **Bulk data on AWS Open Data:** https://registry.opendata.aws/open-meteo/

**Format:** JSON, FlatBuffers (Python client)

**Update frequency:** Forecasts updated every hour; historical data continuously extended.

**Licensing:** Free for open-source and non-commercial use. Commercial API plans available. CC-BY 4.0 for open data.

**Decision support use cases:**
- Short-term renewable generation forecasting (wind and solar)
- Electricity demand forecasting (temperature-driven)
- Heating/cooling degree-day analysis
- Historical backtesting of energy models
- Weather-driven trading strategies

**Key links:**
- Documentation: https://open-meteo.com/en/docs
- Historical API: https://open-meteo.com/en/docs/historical-weather-api
- GitHub: https://github.com/open-meteo/open-meteo

---

### 4.2 Copernicus Climate Data Store (CDS) / ERA5

**What it is:** The EU's flagship climate data service. ERA5 is ECMWF's 5th generation global reanalysis dataset -- the gold standard for historical weather data in energy modeling.

**Data fields (ERA5):**
- **Atmospheric variables:** temperature, wind speed/direction (at surface and pressure levels), humidity, pressure, precipitation, cloud cover
- **Surface solar radiation:** GHI (surface_solar_radiation_downwards), DNI (direct), DHI derivable
- **Energy-specific variables:** wind and solar capacity factors, heating/cooling degree days
- Resolution: 0.25 x 0.25 degrees (~30 km), hourly
- Coverage: global, 1940 to present (5-day latency)
- 37 pressure levels for atmospheric data

**How to collect:**

1. **CDS API (Python):**
   ```bash
   pip install "cdsapi>=0.7.7"
   ```
   - Get Personal Access Token from https://cds.climate.copernicus.eu/profile
   - Configure `~/.cdsapirc`:
     ```
     url: https://cds.climate.copernicus.eu/api
     key: YOUR_PERSONAL_ACCESS_TOKEN
     ```
   ```python
   import cdsapi

   c = cdsapi.Client()
   c.retrieve('reanalysis-era5-single-levels', {
       'product_type': 'reanalysis',
       'variable': [
           '2m_temperature', '10m_u_component_of_wind',
           '10m_v_component_of_wind', 'surface_solar_radiation_downwards',
       ],
       'year': '2024',
       'month': ['01', '02', '03'],
       'day': [str(d).zfill(2) for d in range(1, 32)],
       'time': [f'{h:02d}:00' for h in range(24)],
       'area': [42, 19, 34, 30],  # Greece bounding box [N, W, S, E]
       'data_format': 'netcdf_legacy',
   }, 'era5_greece_2024_q1.nc')
   ```

2. **Google Earth Engine:** ERA5 available as `ECMWF/ERA5/DAILY` and `ECMWF/ERA5_HOURLY`

**Format:** GRIB, NetCDF

**Update frequency:** Daily (5-day latency). ERA5 continuously extended.

**Licensing:** Free with registration. Copernicus licence (CC-BY 4.0 for most products).

**Decision support use cases:**
- High-quality renewable resource assessment
- Long-term climate risk analysis for energy infrastructure
- Training data for ML-based weather/energy forecasting models
- Validation of numerical weather prediction models
- Historical energy system scenario analysis

**Key links:**
- CDS portal: https://cds.climate.copernicus.eu/
- ERA5 dataset: https://cds.climate.copernicus.eu/datasets/reanalysis-era5-single-levels
- API setup guide: https://cds.climate.copernicus.eu/how-to-api

---

### 4.3 ECMWF Open Data (Forecasts)

**What it is:** Free real-time weather forecast data from ECMWF's IFS and AIFS models. ECMWF transitioned to open data on 1 October 2025.

**Data fields:**
- Temperature, wind, precipitation, cloud cover, pressure
- 4 daily forecast cycles (00, 06, 12, 18 UTC)
- IFS: 9 km native resolution; open data subset at 0.25 degree
- AIFS: AI-based forecasting model
- Rolling archive: most recent 12 forecast runs (~2-3 days)

**How to collect:**
```bash
pip install ecmwf-opendata
```
```python
from ecmwf.opendata import Client

client = Client()
result = client.retrieve(
    step=24,
    type="fc",
    param=["2t", "10u", "10v", "ssrd"],
    target="forecast.grib"
)
```

Also available via:
- AWS: https://registry.opendata.aws/ecmwf-forecasts/
- Microsoft Azure Planetary Computer
- Google Cloud Platform
- Open-Meteo (wraps ECMWF open data at full 9 km resolution)

**Licensing:** CC-BY 4.0

**Key links:**
- Open data portal: https://data.ecmwf.int/
- Python package: https://pypi.org/project/ecmwf-opendata/

---

## 5. Carbon & Emissions

### 5.1 EU ETS Data

**What it is:** European Union Emissions Trading System -- the world's largest carbon market. Data on allowance prices, auction results, and installation-level emissions.

**Data fields:**

**Prices and auctions (EEX):**
- EU Allowance (EUA) spot prices (daily)
- EUA futures prices (monthly, quarterly, yearly contracts)
- Auction results: clearing price, volume, bid-to-cover ratio
- Since May 2025: EUA 2 futures (ETS2 for buildings and transport)

**Emissions and allowances (EEA):**
- Verified emissions per installation per year (tCO2)
- Allocated allowances per installation
- Surrendered units
- Coverage: ~15,000 stationary installations + 1,500 aircraft operators across 33 countries

**How to collect:**

1. **EEA Data Viewer:**
   - https://www.eea.europa.eu/en/analysis/maps-and-charts/emissions-trading-viewer-1-dashboards
   - Download CSV from dashboards
   - Also: https://www.eea.europa.eu/en/datahub/datahubitem-view/98f04097-26de-4fca-86c4-63834818c0c0

2. **EEX auction results:**
   - https://www.eex.com/en/markets/environmental-markets/eu-ets-auctions
   - Downloadable auction calendars and results

3. **Datahub.io (structured download):**
   - https://datahub.io/core/eu-emissions-trading-system

4. **Sandbag Carbon Price Viewer:**
   - https://sandbag.be/carbon-price-viewer/ -- historical price chart since 2008 (no longer updated after April 2025, but historical data useful)
   - https://sandbag.be/eu-ets-dashboard/ -- emissions by sector/installation

5. **ICAP Allowance Price Explorer:**
   - https://icapcarbonaction.com/en/ets-prices -- download carbon prices from multiple ETS globally

**Format:** CSV, Excel, PDF (auction results)

**Licensing:** Free and publicly accessible.

**Decision support use cases:**
- Carbon cost integration into power price models
- Marginal abatement cost analysis
- Compliance strategy for industrial emitters
- Portfolio risk management (carbon price exposure)
- Clean spark spread / clean dark spread calculations

---

### 5.2 Plant-Level Emissions (E-PRTR / European Industrial Emissions Portal)

**What it is:** European Pollutant Release and Transfer Register covering ~33,000 industrial facilities across 33 countries, reporting 91 pollutant substances across 65 industrial sub-sectors.

**Data fields:**
- CO2, NOx, SOx, PM and 88 other pollutant releases (to air, water, land)
- Facility location (geocoded), sector, capacity
- Large combustion plant (LCP) specific: energy input, emissions, operating hours
- Annual reporting

**How to collect:**
- Download portal: https://industry.eea.europa.eu/download
- Dashboard: https://industry.eea.europa.eu/industrial-emissions/dashboards

**Format:** CSV, Excel

**Licensing:** Free and open.

**Decision support use cases:**
- Emissions intensity benchmarking
- Carbon leakage risk assessment
- Environmental compliance monitoring
- Industrial decarbonization pathway analysis

---

### 5.3 Electricity Maps

**What it is:** Commercial platform providing real-time carbon intensity of electricity generation for 190+ countries. Open-source data parsers.

**Data fields:**
- Carbon intensity (gCO2eq/kWh) -- lifecycle and direct
- Electricity mix by source (%)
- Power generation and consumption (MWh)
- Import/export flows
- Historical, real-time, and 72-hour forecast
- 1-hour resolution

**How to collect:**

1. **REST API (requires API key, commercial pricing):**
   - `GET https://api.electricitymap.org/v3/carbon-intensity/history?zone=GR`
   - `GET https://api.electricitymap.org/v3/carbon-intensity/past-range?zone=GR&start=2024-01-01T00:00:00Z&end=2024-01-31T00:00:00Z`
   - All endpoints return hourly resolution

2. **Open-source parsers (GitHub):**
   - https://github.com/electricitymaps/electricitymaps-contrib
   - Contains the data collection parsers used by the platform

**Licensing:** Commercial API (paid plans). Open-source parsers under MIT licence.

**Decision support use cases:**
- Carbon-aware computing and scheduling
- Green energy certificate validation
- Corporate Scope 2 emissions reporting
- Demand-side carbon optimization

**Key links:**
- API docs: https://docs.electricitymaps.com/
- Open source: https://github.com/electricitymaps/electricitymaps-contrib

---

## 6. Grid & Infrastructure

### 6.1 Global Power Plant Database (WRI)

**What it is:** Comprehensive open-source database of power plants worldwide from the World Resources Institute. Covers ~35,000 plants in 167 countries.

**Data fields:**
- Plant name, owner/operator
- Geolocation (latitude/longitude)
- Capacity (MW)
- Primary and secondary fuel type
- Generation (GWh, where available)
- Commissioning year
- Source references

**How to collect:**
- Download from GitHub: https://github.com/wri/global-power-plant-database
- CSV format, latest version 1.3.0
- Also available via Google Earth Engine

**Format:** CSV

**Licensing:** CC-BY 4.0

**Important note:** This project is **no longer maintained** (last update 2021). Use as a baseline and supplement with more current data from ENTSO-E or national sources.

**Decision support use cases:**
- Power plant fleet analysis
- Generation capacity mapping
- Infrastructure investment planning
- Geospatial energy analysis

---

### 6.2 Open Power System Data (OPSD)

**What it is:** A free platform dedicated to European power system data. Collects, processes, documents, and publishes publicly available data in convenient formats.

**Data fields (5 data packages):**
1. **Time series:** Electricity consumption, generation by source (wind, solar, other), spot prices -- 15-min/hourly resolution for multiple European countries
2. **Conventional power plants:** Individual plant data (capacity, fuel type, location, commissioning year) for Germany and European countries
3. **Renewable power plants:** Individual installations (capacity, technology, location, tariff eligibility) -- primarily Germany but expanding
4. **National generation capacity:** Installed capacity by country and technology
5. **Household data:** Consumption profiles (limited)

**How to collect:**
- Data platform: https://data.open-power-system-data.org/
- Direct CSV/XLSX/SQLite downloads per data package
- Each package versioned with DOI for citation

**Format:** CSV, XLSX, SQLite

**Licensing:** Various open licences depending on source data.

**Decision support use cases:**
- Power system modeling input data
- Research-grade time series for ML training
- Cross-country energy system comparison

**Key links:**
- Data packages: https://data.open-power-system-data.org/
- Data sources: https://open-power-system-data.org/data-sources

---

## 7. Regulatory & Policy

### 7.1 ACER (EU Agency for Cooperation of Energy Regulators)

**What it is:** EU agency monitoring wholesale energy markets under REMIT regulation. New REMIT Data Reference Centre established May 2025.

**Data fields:**
- Wholesale energy market trading activity
- Inside information disclosures (planned outages, capacity changes)
- Market surveillance data (aggregated)
- Market integration progress reports
- Bidding zone reviews

**How to collect:**
- ACER monitoring reports: https://www.acer.europa.eu/monitoring/electricity_market_integration_2024
- REMIT Data Reference Centre: expected to provide centralized data access (established May 2025, datasets reflecting Q1 2025 trading activity)
- REMIT documents: https://www.acer.europa.eu/remit-documents

**Format:** PDF reports, structured datasets (emerging via REMIT Reference Centre)

**Licensing:** Free and publicly accessible.

**Decision support use cases:**
- Regulatory compliance monitoring
- Market design impact analysis
- Cross-border market integration assessment

---

### 7.2 Eurostat Energy Database

**What it is:** EU's statistical office providing harmonized energy statistics across all Member States.

**Data fields:**
- **Complete energy balances** (`nrg_bal_c`): supply, transformation, final consumption by sector and fuel type
- **Electricity generation** (`nrg_ind_peh`): monthly, by fuel type and country
- **Energy prices:** electricity and gas consumer prices by country and consumption band
- **Renewable energy share** in gross final consumption
- **Imports/exports** of energy products
- Coverage: all EU/EEA/candidate countries

**How to collect:**

1. **Data Browser:** https://ec.europa.eu/eurostat/databrowser/explore/all/all_themes?node_code=nrg_bal_c

2. **Bulk download:** https://ec.europa.eu/eurostat/databrowser/bulk?lang=en

3. **SDMX API:**
   ```
   https://ec.europa.eu/eurostat/api/dissemination/sdmx/2.1/data/nrg_bal_c?format=TSV&compressed=true
   ```

4. **Python package:**
   ```bash
   pip install eurostat
   ```
   ```python
   import eurostat

   # Get table of contents
   toc = eurostat.get_toc_df()

   # Download complete energy balance for Greece
   df = eurostat.get_data_df('nrg_bal_c', filter_pars={'geo': 'EL'})
   ```

**Format:** TSV (API), CSV/XLSX (bulk), Pandas DataFrames (Python)

**Update frequency:** Monthly/quarterly/annual depending on dataset.

**Licensing:** Free. Eurostat open data policy.

**Decision support use cases:**
- National energy planning
- Cross-country benchmarking
- Energy transition monitoring
- Policy impact assessment

**Key links:**
- Energy database: https://ec.europa.eu/eurostat/web/energy/database
- Python package: https://pypi.org/project/eurostat/

---

## 8. Energy Consumption & Efficiency

### 8.1 EU Building Stock Observatory

**What it is:** EU platform tracking the energy performance of buildings across Member States under the EPBD.

**Data fields:**
- Energy Performance Certificate (EPC) statistics by country
- Building stock characteristics (type, age, insulation level)
- Energy consumption per building type
- Renovation rates
- Near-zero energy building (NZEB) statistics

**How to collect:**
- https://energy.ec.europa.eu/topics/energy-efficiency/energy-efficient-buildings/eu-building-stock-observatory_en
- Downloadable indicators per country

**Greece-specific:** The national EPC database is managed under the KENAK regulation. RAAEY oversees compliance. Aggregated/anonymized data is mandated to be publicly available under the revised EPBD (Directive 2024/1275), with Greece currently transposing this directive.

**Decision support use cases:**
- Building energy retrofit prioritization
- National renovation strategy planning
- Green mortgage and real estate valuation

---

### 8.2 Smart Meter Data (Greece/HEDNO)

**Current status:** HEDNO (Hellenic Electricity Distribution Network Operator) is deploying 3.12 million smart meters (2023--2030 rollout). As of late 2025, over 1.1 million meters installed, covering 55%+ of national energy by volume. Meters record consumption at 15-minute intervals.

**Public data availability:** Currently **no public API or open data portal** for HEDNO smart meter data. Data is used internally for grid management and billing. Aggregated demand data available through ADMIE/IPTO (see Section 10).

**Future potential:** As Greece implements EU Electricity Market Design reform and Clean Energy Package requirements, aggregated consumption profiles may become publicly available.

---

## 9. Commodity Prices

### 9.1 EIA (US Energy Information Administration)

**What it is:** The premier source for global oil, gas, and energy commodity data. Free API with extensive historical data.

**Data fields:**
- **Brent crude oil:** spot prices from May 1987 (daily/weekly/monthly/annual)
- **WTI crude oil:** spot prices from February 1986
- **Henry Hub natural gas:** spot prices
- **Coal prices:** various benchmarks
- **Petroleum product prices**
- **US electricity data** (generation, consumption, prices by state)

**How to collect:**

1. **REST API v2:**
   - Base URL: `https://api.eia.gov/v2/`
   - Free API key required: register at https://www.eia.gov/opendata/
   - Example: `https://api.eia.gov/v2/petroleum/pri/spt/data/?api_key=YOUR_KEY&frequency=daily&data[0]=value&facets[series][]=RBRTE&start=2024-01-01`

2. **Python packages:**
   ```bash
   pip install myeia
   # or
   pip install eiapy
   ```
   ```python
   from myeia.api import API

   eia = API(token='YOUR_API_KEY')
   df = eia.get_series(series_id='PET.RBRTE.D')  # Brent crude daily
   ```

**Format:** JSON (API), CSV (bulk download)

**Licensing:** Free with API key registration. US government open data.

**Key links:**
- Open data portal: https://www.eia.gov/opendata/
- API docs: https://www.eia.gov/opendata/documentation.php
- Spot prices page: https://www.eia.gov/dnav/pet/pet_pri_spt_s1_d.htm

---

### 9.2 FRED (Federal Reserve Economic Data)

**What it is:** St. Louis Fed's database of 816,000+ economic time series including key commodity prices.

**Data fields:**
- WTI oil prices (series: `DCOILWTICO`)
- Brent oil prices (series: `DCOILBRENTEU`)
- Natural gas prices Henry Hub (series: `DHHNGSP`)
- Coal prices
- Various economic indicators relevant to energy demand (GDP, industrial production, CPI)

**How to collect:**
```bash
pip install fredapi
```
```python
from fredapi import Fred

fred = Fred(api_key='YOUR_FRED_API_KEY')

# Brent crude oil daily
brent = fred.get_series('DCOILBRENTEU')

# WTI oil daily
wti = fred.get_series('DCOILWTICO')

# Henry Hub natural gas
gas = fred.get_series('DHHNGSP')
```

**Format:** JSON (API), Pandas Series (Python)

**Licensing:** Free with API key (register at https://fred.stlouisfed.org/docs/api/api_key.html)

---

### 9.3 Other Commodity Price APIs

| Source | URL | Oil/Gas? | Free Tier? |
|--------|-----|----------|-----------|
| Oil Price API | https://www.oilpriceapi.com/ | Brent, WTI, TTF | Yes (rate limited) |
| Commodities-API | https://commodities-api.com/ | Oil, gas, metals, agricultural | Yes (limited calls) |
| CommodityPriceAPI | https://commoditypriceapi.com/ | 130+ commodities | Yes |
| Trading Economics | https://tradingeconomics.com/ | Comprehensive | Limited free; paid API |

---

## 10. Greek-Specific Energy Data

### 10.1 ADMIE/IPTO (Independent Power Transmission Operator)

**What it is:** Greece's transmission system operator. Publishes operational data on the Greek electricity system.

**Data fields:**
- **System load:** actual and forecasted load (hourly)
- **Generation mix:** production by fuel type -- lignite, natural gas, hydroelectric, RES, and interconnections balance (hourly and daily)
- **Cross-border physical flows:** imports/exports with Albania, North Macedonia, Bulgaria, Turkey, Italy (hourly)
- **Balancing market data:** ISP results (activated energy, prices)
- **Weighted average market price (WAMP)**
- **Monthly Energy Reports** (comprehensive summaries)

**How to collect:**

1. **File Download API:**
   - Find file URLs: `https://www.admie.gr/getOperationMarketFile?dateStart=2025-01-01&dateEnd=2025-01-31&FileCategory=FILETYPE`
   - Alternative with range overlap: `https://www.admie.gr/getOperationMarketFilewRange?dateStart=2025-01-01&dateEnd=2025-01-31&FileCategory=FILETYPE`
   - Returns JSON with file download URLs
   - File naming convention: `YYYYMMDD_FILETYPE.ext`
   - Known FileCategory types include: `RealTimeSCADASystemLoad`, `ISPWeekAheadLoadForecast`, and ISP-related file types (ISP1, ISP2, ISP3 results/requirements)

2. **Web data views:** https://www.admie.gr/en/market/market-statistics/detail-data

3. **Mobile app:** "IPTO Analytics" for real-time visualization

**Format:** Excel/CSV files (via API), JSON (API metadata)

**Update frequency:** Near real-time for system load; daily for market data.

**Licensing:** Free and publicly accessible.

**Decision support use cases:**
- Greek electricity market analysis
- Load forecasting for Greek system
- Cross-border trading optimization
- Renewable integration monitoring in Greece
- Balancing market participation strategy

**Key links:**
- Main page: https://www.admie.gr/en
- File API: https://www.admie.gr/en/market/market-statistics/file-download-api
- Data views: https://www.admie.gr/en/market/market-statistics/detail-data

---

### 10.2 HEnEx (Hellenic Energy Exchange)

**What it is:** Greece's organized electricity market operator, established 2018. Operates under the EnExGroup umbrella together with EnEx Clearing House.

**Data fields:**
- **Day-Ahead Market (DAM):** market clearing prices (EUR/MWh per hour), aggregated volumes per market time unit and technology portfolio
- **Intraday Market (IDM):** three intraday auctions with separate clearing prices
- **Forward Market:** positions for delivery/offtake and nominations
- **Pre-market data:** forward market positions
- **Post-market data:** anonymous aggregated buy/sell curves, block order results, market statistics

**How to collect:**
- **Web publication:** https://www.enexgroup.gr/markets-publications-el-day-ahead-market
- **APIs:** HEnEx offers APIs for trading management -- access details require registration/membership
- **ENTSO-E mirror:** Greek DAM prices are published on ENTSO-E Transparency Platform (bidding zone `10YGR-HTSO-----Y`) -- this is the easiest programmatic access route

**Format:** Web downloads (PDF/Excel), XML/JSON via ENTSO-E

**Licensing:** Market results are publicly published. Full API access may require market participant status.

**Key links:**
- Main: https://www.enexgroup.gr/
- Energy markets: https://www.enexgroup.gr/energy-markets

---

### 10.3 DESFA (Natural Gas Transmission System Operator)

**What it is:** Greek gas TSO, operating the National Natural Gas Transmission System (NNGS) including the Revithoussa LNG terminal.

**Data fields:**
- **Historical nominations/allocations** (from January 2023 onward)
- **Gas deliveries and off-takes** by entry/exit point
- **LNG terminal data:** inventory, send-out, regasification
- **Transmission tariffs**
- **Yearly NNGS data**
- **System operation reports**

**How to collect:**
- Historical data: https://www.desfa.gr/en/our-services/transmission-services/transmission-user-information/historical-data-on-the-operation-of-the-transmission-system/
- Yearly data: https://www.desfa.gr/en/regulated-services/transmission/pliroforisimetaforas-page/ng-market-data/other-data
- ENTSOG API also covers Greek transmission points (see Section 2.1)

**Format:** Excel files (downloadable from website)

**Licensing:** Free and publicly accessible.

**Decision support use cases:**
- Greek gas supply analysis
- LNG terminal utilization
- Gas-power coupling analysis specific to Greece
- TAP/IGI interconnection flow analysis

---

### 10.4 RAAEY (Regulatory Authority for Energy, Waste and Water)

**What it is:** Greece's independent regulatory authority (formerly RAE, renamed 2023 by Law 5037/2023 with expanded scope).

**Data fields:**
- **GeoPortal:** Geographic distribution of renewable energy infrastructure in Greece (interactive map)
- **Market monitoring reports:** wholesale and retail electricity/gas market analysis
- **Licensing data:** generation and supply licences
- **Consumer complaints data**
- **Natural gas flow data** (via ENTSOG integration): https://www.raaey.gr/energeia/en/natural-gas-flows/

**How to collect:**
- Main website: https://www.raaey.gr/energeia/en/
- GeoPortal for RES infrastructure mapping
- Reports section for market monitoring publications

**Format:** PDFs, interactive maps, web interfaces

**Licensing:** Free and publicly accessible.

---

### 10.5 DAPEEP (RES Operator & Guarantees of Origin)

**What it is:** Non-profit entity managing Greece's renewable energy market on the National Interconnected System, administering the Special Account for RES and CHP, and managing Guarantees of Origin.

**Data fields:**
- **Special Market Prices** by technology (Wind, PV, Hydro, Biomass) -- monthly reference prices in EUR/MWh
- **Feed-in tariff rates** and support scheme details
- **RES generation data** (aggregated)
- **Guarantees of Origin** registry
- **Special Account balance** (RES surcharge revenues vs. support payments)

**How to collect:**
- Official website: https://www.dapeep.gr (primarily in Greek)
- Monthly bulletins with reference prices
- ENTSO-E Transparency Platform mirrors aggregated RES generation for Greece

**Format:** PDF reports, Excel files

**Licensing:** Free and publicly accessible (mostly in Greek).

---

### 10.6 Logariasmo.gr (Third-party Aggregator)

**What it is:** Greek energy data aggregator platform (founded 2024) that consolidates data from RAE and HEnEx for consumer-facing analysis.

**Data sources integrated:** RAE data, HEnEx market prices, supplier tariff data.

**Usefulness:** Demonstrates that programmatic access to Greek energy pricing data is feasible, even if official APIs are limited. The platform parses electricity bills and compares supplier tariffs.

**Key link:** https://www.logariasmo.gr/en/data-sources

---

## 11. Aggregator Platforms & Meta-Sources

### 11.1 Energy-Charts (Fraunhofer ISE)

**What it is:** Free data platform from Fraunhofer Institute for Solar Energy Systems covering 42 European countries. No registration required.

**Data fields:**
- Electricity generation by source (15-min to hourly resolution)
- Day-ahead electricity prices (most European bidding zones)
- Carbon emissions from power generation
- Import/export flows
- Installed capacity
- Forecast data (generation, prices)

**How to collect:**

1. **REST API (free, no registration):**
   - Base URL: `https://api.energy-charts.info/`
   - Prices: `https://api.energy-charts.info/price?bzn=GR`
   - Documentation: https://api.energy-charts.info/ and https://www.energy-charts.info/api.html
   - Returns JSON

**Format:** JSON

**Update frequency:** Real-time (15-min resolution for some data)

**Licensing:** Free, no registration.

**Decision support use cases:**
- Quick access to European electricity prices without ENTSO-E API setup
- Real-time generation mix monitoring
- Price comparison across European markets

**Key links:**
- Web: https://www.energy-charts.info/
- API: https://api.energy-charts.info/

---

### 11.2 Ember Climate / Ember Energy

**What it is:** Independent energy think tank providing open global electricity data. CC-BY-4.0 licensed.

**Data fields:**
- Yearly electricity generation by country and source (223 countries)
- Monthly electricity generation (88 countries)
- Power sector emissions (ktCO2)
- Carbon intensity of electricity (gCO2/kWh)
- Installed capacity (MW)
- Electricity demand and imports

**How to collect:**

1. **REST API:**
   - Request free API key at https://ember-climate.org/data/api
   - Updated twice monthly

2. **CSV bulk download:**
   - Yearly: https://ember-energy.org/data/yearly-electricity-data/
   - Monthly: available from same data page

3. **GitHub:** https://github.com/ember-energy/ember-data-api

**Format:** CSV, JSON (API)

**Licensing:** CC-BY 4.0

**Decision support use cases:**
- Global electricity transition benchmarking
- Carbon intensity tracking
- Country-level energy system analysis

---

### 11.3 Our World in Data -- Energy

**What it is:** Comprehensive, regularly updated dataset covering global energy metrics. Maintained by researchers at the University of Oxford.

**Data fields:**
- Energy consumption by source (primary energy, per capita, growth rates)
- Electricity mix (generation by source)
- Electricity access
- Fossil fuel consumption
- CO2 emissions from energy
- Energy intensity
- Coverage: all countries, annual, varying start dates (some from 1900+)

**How to collect:**
- **Direct downloads:**
  - CSV: https://owid-public.owid.io/data/energy/owid-energy-data.csv
  - XLSX: https://owid-public.owid.io/data/energy/owid-energy-data.xlsx
  - JSON: https://owid-public.owid.io/data/energy/owid-energy-data.json
- **GitHub repository:** https://github.com/owid/energy-data (includes full codebook)
- **CO2 dataset:** https://github.com/owid/co2-data

**Format:** CSV, XLSX, JSON

**Licensing:** CC-BY 4.0

---

## 12. Kaggle & Academic Datasets

### 12.1 Kaggle Datasets

| Dataset | Description | Link |
|---------|-------------|------|
| European Electricity Price & Generation 2024-25 | Recent ENTSO-E data pre-processed | https://www.kaggle.com/datasets/patrikpetovsky/european-electricity-price-and-generation-2024-25 |
| Day-Ahead Electricity Price Forecasting | Price data + LSTM notebook | https://www.kaggle.com/datasets/pythonafroz/day-ahead-electricity-price-forecasting |
| Electricity Spot Price Data | Historical spot prices | https://www.kaggle.com/datasets/arashnic/electricity-spot-price |
| Nord Pool Energy Market Data | Nordic/European market data | https://www.kaggle.com/datasets/pythonafroz/nord-pool-energy-market-data |
| European Countries Electrical Power Trading Data | Cross-border trading | https://www.kaggle.com/datasets/pythonafroz/european-countries-electrical-power-trading-data |
| Electricity Load Forecasting | Load time series | https://www.kaggle.com/datasets/saurabhshahane/electricity-load-forecasting |
| Energy Consumption Prediction | Consumption forecasting | https://www.kaggle.com/datasets/mrsimple07/energy-consumption-prediction |
| U.S. Electricity Prices | US price data | https://www.kaggle.com/datasets/alistairking/electricity-prices |

### 12.2 Academic / Institutional Datasets

| Source | Description | Link |
|--------|-------------|------|
| ENTSO-E ERAA Modelling Data | European Resource Adequacy Assessment input data (demand projections, capacity, RES profiles) | https://www.entsoe.eu/eraa/2025/modelling-data/ |
| ENTSO-E Power Statistics | Official statistical factsheets | https://www.entsoe.eu/data/power-stats/ |
| JRC publications on EU natural gas | Tools and data for analyzing the European gas system | https://publications.jrc.ec.europa.eu/repository/bitstream/JRC130771/JRC130771_01.pdf |
| Open Energy Modelling Initiative Wiki | Curated list of open data sources for energy modeling | https://wiki.openmod-initiative.org/wiki/Data |

---

## 13. Open Source Energy Models with Data

### 13.1 PyPSA (Python for Power System Analysis)

**What it is:** Open-source Python framework for optimizing and simulating modern power and energy systems. MIT licence.

**Capabilities:** Conventional generators with unit commitment, variable wind/solar, hydro, storage, sector coupling, elastic demands, DC/AC linearized power flow.

**Installation:**
```bash
pip install pypsa
```

**Data-integrated models:**
- **PyPSA-Eur:** Full ENTSO-E area sector-coupled model. Downloads and processes ENTSO-E data, OpenStreetMap network topology, weather data. https://github.com/PyPSA/pypsa-eur
- **PyPSA-Earth:** Global cross-sectoral model with high resolution. https://github.com/pypsa-meets-earth/pypsa-earth

**Key link:** https://pypsa.org/

---

### 13.2 Atlite

**What it is:** Lightweight Python package for converting weather data into renewable energy time series (wind power, solar PV, solar thermal, hydro, heating demand). Used by PyPSA-Eur.

**How it works:** Creates "cutouts" (spatio-temporal subsets) of weather data, then converts to capacity factors using turbine/panel models.

**Installation:**
```bash
pip install atlite
```
```python
import atlite

# Create cutout for Greece using ERA5
cutout = atlite.Cutout(
    path="greece-2024",
    module="era5",
    x=slice(19, 30),  # longitude
    y=slice(34, 42),  # latitude
    time="2024"
)
cutout.prepare()

# Calculate wind capacity factors
wind_cf = cutout.wind(turbine="Vestas_V112_3MW")

# Calculate solar PV capacity factors
solar_cf = cutout.pv(panel="CSi", orientation="latitude_optimal")
```

**Data source:** Automatically retrieves ERA5 data from Copernicus CDS API.

**Key link:** https://github.com/PyPSA/atlite

---

### 13.3 pvlib Python

**What it is:** Community-maintained toolbox for simulating photovoltaic energy systems. Originally from Sandia National Laboratories.

**Capabilities:** Solar position, clear sky irradiance, irradiance transposition, PV module/inverter modeling, temperature models, DC/AC power conversion. Includes built-in data retrieval from PVGIS, NASA POWER, and other sources.

**Installation:**
```bash
pip install pvlib
```

**Key link:** https://github.com/pvlib/pvlib-python

---

## 14. Python Packages Summary

| Package | Purpose | Install | API Key? |
|---------|---------|---------|----------|
| `entsoe-py` | ENTSO-E electricity market data | `pip install entsoe-py` | Yes (free) |
| `entsog-py` | ENTSOG gas transmission data | `pip install entsog-py` | No |
| `gie-py` | GIE gas storage (AGSI+) and LNG (ALSI) | `pip install gie-py` | Yes (free) |
| `openmeteo-requests` | Open-Meteo weather data | `pip install openmeteo-requests` | No |
| `cdsapi` | Copernicus Climate Data Store (ERA5) | `pip install cdsapi` | Yes (free) |
| `ecmwf-opendata` | ECMWF open forecast data | `pip install ecmwf-opendata` | No |
| `pvlib` | PV system simulation + data retrieval (PVGIS) | `pip install pvlib` | No |
| `atlite` | Weather-to-energy time series conversion | `pip install atlite` | CDS key needed |
| `pypsa` | Power system analysis/optimization | `pip install pypsa` | No |
| `eurostat` | Eurostat statistical data | `pip install eurostat` | No |
| `fredapi` | FRED economic/commodity data | `pip install fredapi` | Yes (free) |
| `myeia` | EIA energy/commodity data | `pip install myeia` | Yes (free) |
| `nordpool` | Nord Pool Elspot prices | `pip install nordpool` | No |

---

## Pricing Data Summary

Energy pricing data is distributed across the sections above. This summary consolidates all pricing sources by commodity type for quick reference.

### Wholesale Electricity Prices

| Source | Coverage | Granularity | History | Access | Section |
|---|---|---|---|---|---|
| **ENTSO-E** | All EU bidding zones including Greece (`10YGR-HTSO-----Y`) | Hourly / 15-min | 2015+ | Free API, `pip install entsoe-py` | 1.1 |
| **Energy-Charts** (Fraunhofer ISE) | 42 European countries | Hourly | 2015+ | Free, no registration. `https://api.energy-charts.info/price?bzn=GR` | 11.1 |
| **HEnEx** | Greek DAM and intraday auctions | Hourly | 2020+ | Easiest via ENTSO-E; direct access may require market participant status | 1.5 |
| **ADMIE/IPTO** | Greek system marginal price, balancing market prices | Hourly | Varies | File download API | 10.1 |
| **Nord Pool** | Nordic, Baltic, and expanding European markets | Hourly | 2012+ | Public portal + `pip install nordpool` | 1.3 |
| **OPSD** | Multiple EU countries (research-grade historical) | Hourly | 2005--2020 | Bulk CSV at `data.open-power-system-data.org` | 6.2 |

### Natural Gas Prices

| Source | What | Granularity | History | Access | Section |
|---|---|---|---|---|---|
| **World Bank Pink Sheet** | TTF (European benchmark), Henry Hub, Japan LNG | Monthly | 1960+ | Free Excel at `worldbank.org/en/research/commodity-markets` | 9.3 |
| **EEX** | TTF Neutral Gas Price index | Daily | Rolling 60 days | Free download at `eex.com` | 2.3 |
| **ICE** | TTF spot and futures (official exchange) | Daily | Full history | Free delayed data; real-time requires subscription | 2.3 |
| **Oil Price API** | TTF, Henry Hub | Daily | Varies | Free tier (rate limited) at `oilpriceapi.com` | 2.3 |
| **FRED** | Henry Hub (`DHHNGSP`) | Daily | 1997+ | Free, `pip install fredapi` | 9.2 |
| **EIA** | Henry Hub + international gas benchmarks | Daily | 1997+ | Free, `pip install myeia` | 9.1 |
| **Commodities-API** | TTF and other gas benchmarks | Daily | 2023+ | Free tier at `commodities-api.com` | 2.3 |

**Gap:** Daily TTF prices with full history are not freely available. The World Bank provides monthly TTF; EEX provides 60 days; Oil Price API and Commodities-API have rate-limited free tiers. Serious gas price modeling at daily resolution requires a commercial exchange data subscription (ICE or EEX).

### Carbon Prices (EU ETS)

| Source | What | Granularity | History | Access | Section |
|---|---|---|---|---|---|
| **EEX** | EUA primary auction clearing prices, volumes, bid-to-cover | Per auction | 2005+ | Free download at `eex.com/en/markets/environmental-markets` | 5.1 |
| **ICAP Allowance Price Explorer** | Carbon prices from multiple global ETS | Daily | 2005+ | Free at `icapcarbonaction.com/en/ets-prices` | 5.1 |
| **Sandbag** | EU ETS price viewer | Daily | 2008--2025 | Free (no longer updated after April 2025) | 5.1 |
| **Datahub.io** | Structured EU ETS download | Daily | 2005+ | Free at `datahub.io/core/eu-emissions-trading-system` | 5.1 |

### Oil & Coal Prices

| Source | What | Granularity | History | Access | Section |
|---|---|---|---|---|---|
| **EIA** | Brent, WTI, petroleum products, coal | Daily / weekly / monthly | 1986+ | Free API, `pip install myeia` | 9.1 |
| **FRED** | Brent (`DCOILBRENTEU`), WTI (`DCOILWTICO`) | Daily | 1986+ | Free, `pip install fredapi` | 9.2 |
| **World Bank Pink Sheet** | Brent, WTI, Dubai crude, Australian/South African coal | Monthly | 1960+ | Free Excel download | 9.3 |
| **Commodities-API** | 130+ commodities (oil, gas, metals, agricultural) | Daily | Varies | Free tier at `commodities-api.com` | 9.3 |

### Retail / Consumer Energy Prices

| Source | What | Granularity | History | Access | Section |
|---|---|---|---|---|---|
| **Eurostat** | Household electricity (`nrg_pc_204`), non-household electricity (`nrg_pc_205`), household gas (`nrg_pc_202`) -- all EU countries by consumption band | Semi-annual | 2007+ | Free, `pip install eurostat` | 7.2 |
| **RAAEY** (Greek regulator) | Greek supplier tariff approvals | Per decision | Varies | Website downloads | 10.4 |

### Renewable Energy Prices

| Source | What | Granularity | History | Access | Section |
|---|---|---|---|---|---|
| **DAPEEP** | Greek RES reference prices by technology (wind, PV, hydro, biomass) in EUR/MWh | Monthly | Varies | Website downloads at `dapeep.gr` | 10.5 |
| **IRENA** | Global LCOE by technology (solar PV, wind, hydro, biomass, geothermal, CSP) | Annual | 2010+ | Free at `irena.org/Data` | 3.3 |

### Forward / Futures Curves

Forward price data is the weakest area for free access. Exchange-traded futures (ICE, EEX, Nasdaq) are commercial products.

| Source | What | Free Access |
|---|---|---|
| **EEX** | Power, gas, carbon futures | 60-day rolling history only |
| **ICE** | TTF gas, Brent oil, EUA carbon futures | Delayed end-of-day data |
| **Trading Economics** | Broad commodity forward curves | Limited free tier |
| **ENTSO-E** | N/A -- spot market only, no forward prices | -- |

For backtesting, FRED and EIA provide sufficient spot price history. For forward curve modeling or mark-to-market of derivatives, a commercial exchange data subscription is required.

### Quickest Path to a Multi-Commodity Price Dataset

```python
from entsoe import EntsoePandasClient
from fredapi import Fred
import eurostat

# Electricity -- Greek day-ahead prices
entsoe_client = EntsoePandasClient(api_key='ENTSOE_TOKEN')
elec_prices = entsoe_client.query_day_ahead_prices('GR', start=start, end=end)

# Natural gas -- Henry Hub daily (TTF requires commercial source for daily)
fred = Fred(api_key='FRED_KEY')
gas_prices = fred.get_series('DHHNGSP')

# Oil -- Brent crude daily
brent = fred.get_series('DCOILBRENTEU')

# Carbon -- download EU ETS from ICAP or Datahub.io, then:
import pandas as pd
carbon = pd.read_csv('eu_ets_prices.csv', parse_dates=['date'], index_col='date')

# Retail electricity prices -- Greek non-household from Eurostat
retail = eurostat.get_data_df('nrg_pc_205', filter_pars={'geo': 'EL'})
```

---

## Summary: Quick Start for a Greek Energy Decision Support System

For a system focused on the Greek energy market, the minimum viable data stack would be:

1. **Electricity prices and generation:** ENTSO-E Transparency Platform via `entsoe-py` (bidding zone `GR`, code `10YGR-HTSO-----Y`)
2. **Greek grid operations:** ADMIE/IPTO File Download API for detailed load, generation mix, and cross-border flows
3. **Gas data:** ENTSOG API for Greek gas flows + GIE AGSI+/ALSI for storage and LNG (Revithoussa)
4. **Weather for forecasting:** Open-Meteo (free, no key) or ERA5 via `cdsapi` (higher quality, registration required)
5. **Solar resource:** PVGIS API for site-specific PV analysis
6. **Carbon prices:** EEX/Sandbag for EU ETS prices; EEA for plant-level emissions
7. **Commodity prices:** EIA API or FRED for oil/gas benchmarks
8. **Quick European overview:** Energy-Charts API (free, no registration)

All of these can be accessed programmatically in Python with the packages listed in Section 14.
