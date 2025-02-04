# Supply-Demand Forecasting System

## Business Problem
Manufacturing and e-commerce company experiencing inventory management challenges including stockouts and excess inventory. Needed to optimize supply chain operations through better demand forecasting and inventory management.

## Solution Approach
### Data Collection & Preparation
- Consolidated data from:
  - Historical sales data
  - Inventory levels
  - Manufacturing schedules
  - Supplier lead times
- Created unified supply chain database

### Analysis & Implementation
1. Demand Analysis
   - Implemented moving average forecasting
   - Created seasonal index calculations
   - Developed promotion impact analysis

2. Inventory Optimization
   - Built safety stock calculator
   - Implemented reorder point optimization
   - Created stock level monitoring system

3. Dashboard Development
   - Developed real-time inventory dashboard
   - Created forecast accuracy tracking
   - Implemented alert system

## Technical Implementation
- **Languages & Tools:**
  - Python (pandas) for analysis
  - SQL for data management
  - Power BI for visualization
- **Key Features:**
  - Automated inventory tracking
  - ABC analysis
  - Safety stock calculation

## Results & Impact
- Reduced stockouts by 35%
- Decreased holding costs by 20%
- Improved forecast accuracy by 25%
- Enhanced supplier relationship management

## Visualizations
![Demand Forecast Dashboard](/images/demand-forecast.png)
![Inventory Management](/images/inventory-management.png)

## Code Snippets
```python
# Example of demand forecasting using moving averages
def calculate_forecast(df):
    # Calculate 3-month moving average
    df['forecast_ma3'] = df['sales'].rolling(window=3).mean()
    
    # Calculate 6-month moving average
    df['forecast_ma6'] = df['sales'].rolling(window=6).mean()
    
    # Calculate seasonal indices
    df['month'] = df['date'].dt.month
    seasonal_indices = df.groupby('month')['sales'].mean() / df['sales'].mean()
    
    return df, seasonal_indices

# Example of ABC analysis
def perform_abc_analysis(df):
    # Calculate total value for each product
    df['total_value'] = df['annual_usage'] * df['unit_cost']
    
    # Sort products by total value
    df = df.sort_values('total_value', ascending=False)
    
    # Calculate cumulative percentage
    df['cumulative_value'] = df['total_value'].cumsum() / df['total_value'].sum()
    
    # Assign ABC categories
    df['category'] = 'C'
    df.loc[df['cumulative_value'] <= 0.8, 'category'] = 'A'
    df.loc[(df['cumulative_value'] > 0.8) & (df['cumulative_value'] <= 0.95), 'category'] = 'B'
    
    return df
```

## Future Enhancements
- Enhanced seasonality analysis
- Advanced inventory optimization
- Integration with ERP system
