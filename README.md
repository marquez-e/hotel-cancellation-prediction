# Hotel booking cancellation and overbooking optimizer

![Python](https://img.shields.io/badge/Python-3.9+-blue.svg)
![FastAPI](https://img.shields.io/badge/FastAPI-0.95+-009688.svg?logo=fastapi&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?logo=postgresql&logoColor=white)
![XGBoost](https://img.shields.io/badge/XGBoost-Machine_Learning-orange.svg)
![Power BI](https://img.shields.io/badge/Power_BI-F2C811?logo=powerbi&logoColor=black)

An end-to-end data and machine learning solution designed to optimize strategic decisions in the hotel sector. This project minimizes the financial risk of empty rooms and the high costs of guest relocation by predicting booking cancellations and recommending optimal overbooking thresholds.

## Summary
Cancellation management and overbooking policies are critical challenges in the hospitality industry. To maximize revenue, hotels must find the perfect financial balance between unsold inventory and relocation penalties. 

This project tackles the problem by taking a machine learning model from raw data to a simulated production environment.

The pipeline includes:
1. **Data engineering.** Automated ETL feeding a PostgreSQL Star Schema.
2. **Model experimentation.** Training and evaluation of baseline and ensemble models (Logistic Regression, Random Forest, XGBoost).
3. **Model deployment.** Encapsulating the best-performing model into a REST API (FastAPI) for real-time serving.
4. **Data prescriptive analytics.** Applying operations research (Newsvendor Problem) and explainable AI (SHAP) on a Power BI dashboard to drive business decisions.

## Machine Learning and architecture stack
* **Data Science and ML:** Scikit-Learn, XGBoost, Random Forest, SHAP (Explainability), Pandas, NumPy, SQLAlchemy
* **ML engineering and serving:** FastAPI, Uvicorn, Docker.
* **Data engineering:** PostgreSQL (Star Schema), Python, ETL.
* **Data visualization and BI:** Power BI, Matplotlib, Seaborn.

## Machine Learning lifecycle and features
* **Data preprocessing and feature engineering.** Treats frequent anomalies, handles missing values, and calculates new variables. Prevents data leakage by ordering datasets by arrival date and lead time, splitting data chronologically (80% training, 20% for future prediction simulation).
* **Model experimentation and selection.** Comparative analysis using **Logistic Regression**, **Random Forest**, and **XGBoost**. XGBoost was selected as the optimal algorithm due to its better performance on highly imbalanced cancellation data, evaluated via the **AUC-PR** metric.
* **Explainable AI.** Integrated SHAP values to decode the "black-box" model.
* **Real-time inference simulator.** Built a Property Management System (PMS) simulator that streams the remaining 20% of unseen test data to the FastAPI endpoints, emulating a production environment.

## Data source
This project uses the [Hotel Booking Demand dataset](https://www.kaggle.com/datasets/jessemostipak/hotel-booking-demand) by Jesse Mostipak. Download it and place the CSV files inside the /data folder before running the ETL.

## Repository structure

```
├── api/                  # FastAPI endpoints and API containerization
├── src/                  # Python scripts for ETL, model training and evaluation and PMS simulator
├── data/                 # Raw CSV file with data from Kaggle
├── dashboard/            # Power BI file
├── graphics /            # Confusion matrix, ROC and PR curves
├── requirements.txt      # Python dependencies
├── .gitignore            # Ignored files and folders
├── .init_schema.sql      # SQL script for PostgreSQL Star Schema creation
├── .env.example          # .env file with variables
└── README.md             # Project documentation  
```

## Local setup & installation

**1. Clone the repository and install dependencies**
```bash
git clone https://github.com/your-username/hotel-overbooking-optimizer.git
cd hotel-overbooking-optimizer
python -m venv venv
source venv/bin/activate  # On Windows use: venv\Scripts\activate
pip install -r requirements.txt
```

**2. Database setup**
* Ensure PostgreSQL is installed and running.
* Create a new database named `hotel_dwh`.
* Run the SQL script located in `dwh/init_schema.sql` to generate the Star Schema structure with all Primary and Foreign Keys:
```bash
psql -U your_user -d hotel_dwh -f dwh/init_schema.sql
```
* Set your environment variables as in .env.example (substitute '...' with the correct values)

**3. Run the ML pipeline (ETL and training)**  
Process the raw data, train the model, and populate the PostgreSQL data warehouse (make sure your raw Kaggle datasets are placed inside the /data folder before running this step):  
```bash
python src/etl.py
```
Next, train the XGBoost model using the 80% historical data slice and generate the trained model file:
```bash
python src/model.py
```

**4. Power BI initial setup**
Before starting the real-time simulation, connect the dashboard to your local database:
1. Open `dashboard/dashboard.pbix` in Power BI Desktop.
2. Go to **Home** > **Transform data** > **Data source settings**.
3. Select the PostgreSQL connection, click **Change Source**, and update the server and database name (e.g., `localhost` and `hotel_dwh`).
4. Edit Permissions to input your local PostgreSQL username and password.
5. Click **Close & Apply**. You will now see the historical data baseline.

**5. Start the services (requires 2 terminals)**
Now, simulate the hotel's Property Management System (PMS) receiving new future bookings and the API predicting their cancellation risk on the fly. Open two separate terminals:

* Terminal 1: start the REST API
```bash
cd api
python main.py
```
or
```bash
uvicorn main:app --reload
```
*(The API will load the trained XGBoost model and listen on http://localhost:8000)*

* Terminal 2: start the PMS simulator
```bash
cd src
python pms_simulator.py
```
*(The simulator will start sending the 20% future data slice to the API).*

**6. What to expect now**  
Once both the API and the PMS Simulator are running, you will see the full end-to-end pipeline working:
* **Terminal 1 (API):** Will log incoming `POST` requests in real-time as the simulator sends new booking data.
* **Terminal 2 (simulator):** Will display logs of the simulated future bookings being processed, showing the model's response (e.g., cancellation probabilities and SHAP reasons).
* **Database:** The `fact_future_simulation` table will be continuously updated with new rows, including the newly calculated `cancellation_probability`, `id_reason_1`, and `id_reason_2`.
* **Power BI:** Hitting "Refresh" will load the newly simulated predictions from PostgreSQL, updating the visuals and the overbooking recommendations dynamically.

## About
This project was developed as a Project for a Bachelor's in Computer Engineering, aiming to demonstrate a Machine Learning Engineering skill set: from data ingestion and model selection to production serving and business intelligencee.