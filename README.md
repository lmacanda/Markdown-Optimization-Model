# Markdown-Optimization-Model
This project integrates large-scale transactional label data with structural store metadata to predict the success of markdown strategies. It features a robust data merging pipeline that contextualizes 150k+ product events with store-level characteristics like size and location to drive optimized pricing decisions
# How to use this project

Run the Pink_Label_exploratory.ipynb to generate the Pink_Label_Master.csv.

Use the Master CSV for training the predictive algorithms or visualizing results in Power BI.

## Goal of the Model

The primary objective of this model is to optimize the Markdown Program for products close to expiration (Pink Labels). By analyzing historical sales data, store characteristics, and product attributes, the model aims to:

- **Reduce Waste**: Identify the optimal timing and discount depth to sell products before they reach their shelf-retrieval date.
- **Maximize Recovery**: Balance the trade-off between margin loss (from discounts) and total loss (from product expiration).

## Data Integration: Merging Databases
The project relies on two primary data sources that capture different dimensions of the retail operation:

**Labels Database** (Data_labels.xlsx): A large transactional-level dataset (150,054 records) containing specific "Pink Label" events, including product SKUs, brands, original pricing, applied discounts, and the eventual sale status.

**Stores Database** (Data_store.xlsx): A smaller structural dataset (342 records) providing context for each physical location, such as store type (e.g., "Large"), selling square footage, and geographic district.


## Data Preprocessing & Cleaning
To ensure the quality of the model's variables the following preprocessing steps were performed:

**✓** Duplicates
The dataset was scanned for redundant transaction or label records to prevent over-representing specific sales events.

**✓** Formats and Normalization
Date Conversion: expiring_date, labelling_date, and sell_date were converted into standardized datetime formats to allow for time-series analysis.

String Cleaning: Brand names and categorical variables were normalized (e.g., removing extra spaces like "Marca 1" vs "Marca 1") to ensure consistent grouping.

Numerical Extraction: Discounts were extracted from string formats (e.g., extracting "0.50" from "2.11 (0.50)") to create usable numeric features.

**✓** Missing Values
Store Metrics: Missing values in selling_square_ft were handled using a hierarchical approach: filling with the median of the specific store type, and falling back to the global median where necessary.

Target Variables: Missing values in the sold status were identified and addressed to ensure a clean binary target for training.

Newpvp and Discount: Dropped 28 rows that represent only 0.018% of the data.

Weight: 0.29% of missing data, 424 rows have been filled by use thes SKU Median of the exact product and bu using the "neighbor data"

**✓** Scales
Numerical features like weight (g) and oldpvp (original price) were reviewed for scaling consistency, ensuring that large units (grams) do not disproportionately influence the model compared to smaller price units.

**✓** Outliers
Extreme values in sales volume and store square footage were identified to prevent rare anomalies (e.g., very high extreme values) from skewing the model's generalizability. 
price over 200€ for a single item has been considered data entry error (like a missing decimal point) than a real product.

## Feature Engineering: New Variables

Beyond the raw data, several high-impact variables were engineered to capture business performance levels:

- **Brand Success Tier**: A categorical variable created by grouping brands based on their historical sales performance.
- **SKU Success Tier**: Similar to brands, individual SKUs were ranked into performance tiers based on their mean sales.
- **Store Tier**: Stores were grouped into three performance levels (Tiers 1, 2, and 3) using a quantile cut (qcut) based on their average sales volume.
- **Days to Expiration**: Derived from the difference between labelling_date and expiring_date to represent the "urgency" of the sale.
- **Discount Percentage**: Extracted and calculated to serve as a primary predictor of the consumer's likelihood to purchase.
- **Price Per Gram** To see if heavier, cheaper items sell differently than small premium ones.

- ## Key Insights (Business Value)

**The Store Factor**: "Store tiers (High/Medium/Low success) were the strongest predictors of clearing inventory."

**Winning Model**: "Logistic Regression achieved an ROC AUC of 0.645, providing the best balance between predictive power and business explainability."

## Current Training Data State

- **Variables Removed**: brand, labelling_date, sell_date, days_to_sell
- **Target**: sold (1 or 0).
- **Features Included**:
*sku* (The unique product identifier).
*Price Metrics*: oldpvp, new_pvp, discount_rate, price_per_gram.
*Context*: type (Store size 1-3), day_of_the_week.
*Location*: 17 District columns (e.g., district_Lisboa).
*Product Stats*: weight, margin, perc_expiring_sku.

# Model Tested

## 🤖 The Machine Learning Pipeline
I implemented a bifurcated pipeline to ensure every algorithm received the data format it required to perform optimally:

- Tree-Based Models (Random Forest, HistGradientBoosting): Trained on raw numerical data to preserve the original decision boundaries of categories like Tiers.

- Distance/Linear Models (KNN, Logistic Regression): Trained on Standardized (Scaled) data to prevent larger numerical values (like price) from skewing the results.


> [!NOTE]

> The Train-Test Split
The model has been trained on 80% of the data and then tested on the 20%

## 📊 Model Comparison & Results

<img width="1600" height="933" alt="image" src="https://github.com/user-attachments/assets/e45c15a2-eda8-405b-9ff7-36b3a436a05e" />

## The Winner: HistGradientBoosting
While accuracy is a standard metric, for food waste management, Recall is the primary focus. HistGradientBoosting, identifies 84.6% of waste cases, allowing retailers to intervene before the product is lost. This is significantly more effective than traditional logistic models, which only captured 62% of potential waste

## 📈 Key Insights

- Store Tier Impact: Stores in "Tier 3" have a significantly higher waste rate regardless of the brand, suggesting operational issues rather than product issues.

- The 50% Threshold: A 50% discount is the "sweet spot" for most brands; increasing the discount further provides diminishing returns on sale probability.

<img width="1600" height="960" alt="image" src="https://github.com/user-attachments/assets/470382e2-db13-4946-9807-ad57a53c6057" />

## "Understanding the Drivers of Waste"

- Store Tier: Highlights that store location and management efficiency are major predictors of stock recovery.

- New PVP: Confirms that pricing is a primary lever for moving nearing-expiry stock.

- SKU & Brand Scores: Show that customer loyalty and product category play a significant role in the success of the "Pink Label" program.


## 🔧 Technical Stack
- Language: Python 3.x

- Libraries: Pandas, NumPy, Scikit-Learn, Matplotlib, Seaborn

- Environment: Jupyter Notebook / Google Colab


  
