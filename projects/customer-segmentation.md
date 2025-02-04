# Customer Segmentation & Return Rate Optimization

## Business Problem
E-commerce company facing challenges with high return rates and inefficient marketing spend due to lack of customer segmentation. Needed to optimize marketing strategies and reduce return rates while maintaining customer satisfaction.

## Solution Approach
### Data Collection & Preparation
- Integrated data from multiple sources:
  - Customer transaction history
  - Product returns data
  - Customer service interactions
  - Marketing campaign data
- Created unified customer database

### Analysis & Implementation
1. Customer Segmentation
   - Implemented RFM (Recency, Frequency, Monetary) analysis
   - Applied k-means clustering to identify distinct customer segments
   - Created customer value scoring system

2. Return Rate Analysis
   - Developed return rate tracking by customer segment
   - Identified high-risk product categories
   - Implemented customer return propensity scoring

3. Dashboard Development
   - Created interactive Power BI dashboard for tracking KPIs
   - Implemented drill-down capabilities for detailed analysis
   - Set up automated data refresh pipeline

## Technical Implementation
- **Languages & Tools:**
  - Python (pandas, sklearn) for analysis
  - SQL for data extraction
  - Power BI for visualization
- **Key Features:**
  - RFM scoring system
  - Basic k-means clustering
  - Automated reporting

## Results & Impact
- 25% reduction in return rates
- Improved marketing ROI through targeted campaigns
- Enhanced customer satisfaction scores
- Automated reporting saving 10+ hours weekly

## Visualizations
![Customer Segmentation Dashboard](/images/customer-segmentation.png)
![Return Rate Analysis](/images/return-analysis.png)

## Code Snippets
```python
# Example of RFM analysis implementation
def calculate_rfm_scores(df):
    # Calculate Recency
    current_date = df['purchase_date'].max()
    df['recency'] = (current_date - df['last_purchase_date']).dt.days
    
    # Calculate Frequency
    frequency_df = df.groupby('customer_id')['order_id'].count().reset_index()
    frequency_df.columns = ['customer_id', 'frequency']
    
    # Calculate Monetary
    monetary_df = df.groupby('customer_id')['total_amount'].sum().reset_index()
    monetary_df.columns = ['customer_id', 'monetary']
    
    # Combine all metrics
    rfm_df = pd.merge(frequency_df, monetary_df, on='customer_id')
    rfm_df = pd.merge(rfm_df, df[['customer_id', 'recency']].drop_duplicates(), on='customer_id')
    
    # Score each component (1-5 scale)
    rfm_df['r_score'] = pd.qcut(rfm_df['recency'], q=5, labels=[5,4,3,2,1])
    rfm_df['f_score'] = pd.qcut(rfm_df['frequency'], q=5, labels=[1,2,3,4,5])
    rfm_df['m_score'] = pd.qcut(rfm_df['monetary'], q=5, labels=[1,2,3,4,5])
    
    return rfm_df

# Example of basic customer segmentation using k-means
def segment_customers(rfm_df):
    from sklearn.preprocessing import StandardScaler
    from sklearn.cluster import KMeans
    
    # Prepare data for clustering
    features = ['r_score', 'f_score', 'm_score']
    X = rfm_df[features].values
    
    # Scale the features
    scaler = StandardScaler()
    X_scaled = scaler.fit_transform(X)
    
    # Apply k-means clustering
    kmeans = KMeans(n_clusters=4, random_state=42)
    rfm_df['segment'] = kmeans.fit_predict(X_scaled)
    
    # Label segments based on characteristics
    segment_labels = {
        0: 'Loyal Customers',
        1: 'At Risk Customers',
        2: 'Lost Customers',
        3: 'New Customers'
    }
    rfm_df['segment_label'] = rfm_df['segment'].map(segment_labels)
    
    return rfm_df

# Example of return rate analysis by segment
def analyze_returns(df):
    return_analysis = df.groupby('segment_label').agg({
        'order_id': 'count',
        'is_returned': 'sum',
        'total_amount': 'sum'
    }).reset_index()
    
    return_analysis['return_rate'] = (
        return_analysis['is_returned'] / return_analysis['order_id']
    )
    return_analysis['avg_order_value'] = (
        return_analysis['total_amount'] / return_analysis['order_id']
    )
    
    return return_analysis
```

```sql
-- Example of customer cohort analysis
WITH cohort_items AS (
    SELECT
        customer_id,
        MIN(DATE_TRUNC('month', first_purchase_date)) as cohort_month,
        COUNT(DISTINCT order_id) as total_orders,
        SUM(total_amount) as total_spend
    FROM customer_orders
    GROUP BY customer_id
),
cohort_size AS (
    SELECT 
        cohort_month,
        COUNT(DISTINCT customer_id) as num_customers,
        SUM(total_orders) as total_orders,
        SUM(total_spend) as total_revenue
    FROM cohort_items
    GROUP BY cohort_month
),
retention_table AS (
    SELECT 
        c.cohort_month,
        DATE_PART('month', o.order_date) - 
        DATE_PART('month', c.cohort_month) as months_since_first_purchase,
        COUNT(DISTINCT o.customer_id) as num_customers
    FROM cohort_items c
    LEFT JOIN orders o ON c.customer_id = o.customer_id
    GROUP BY 1, 2
)
SELECT 
    cohort_month,
    months_since_first_purchase,
    num_customers,
    ROUND(100.0 * num_customers / 
        FIRST_VALUE(num_customers) OVER (
            PARTITION BY cohort_month ORDER BY months_since_first_purchase
        ), 2) as retention_rate
FROM retention_table
ORDER BY cohort_month, months_since_first_purchase;
```

## Future Enhancements
- Enhanced segmentation with additional customer attributes
- Advanced return prediction analytics
- Integration with marketing automation platforms
