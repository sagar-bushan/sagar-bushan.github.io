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
   - Developed composite scoring system based on purchase patterns
   - Implemented dynamic segmentation using natural breaks
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
  - Python (pandas) for analysis
  - SQL for data extraction
  - Power BI for visualization
- **Key Features:**
  - Advanced customer segmentation
  - Dynamic scoring system
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
def segment_customers(df):
    # Calculate core customer metrics
    customer_metrics = df.groupby('customer_id').agg({
        'order_id': 'count',
        'total_amount': 'sum',
        'last_purchase_date': 'max',
        'first_purchase_date': 'min'
    }).reset_index()
    
    # Derive customer behavior metrics
    customer_metrics['avg_order_value'] = customer_metrics['total_amount'] / customer_metrics['order_id']
    
    # Calculate purchase timespan and frequency
    customer_metrics['purchase_span'] = (customer_metrics['last_purchase_date'] - 
                                     customer_metrics['first_purchase_date']).dt.days
    
    # Handle edge case of single-day purchasers
    customer_metrics['purchase_span'] = customer_metrics['purchase_span'].replace(0, 1)
    customer_metrics['purchase_frequency'] = customer_metrics['order_id'] / customer_metrics['purchase_span']
    
    # Calculate percentile rankings for key metrics
    for col in ['purchase_frequency', 'avg_order_value', 'total_amount']:
        customer_metrics[f'{col}_rank'] = customer_metrics[col].rank(pct=True)
    
    # Composite score calculation with empirically determined weights
    customer_metrics['value_score'] = (
        0.4 * customer_metrics['total_amount_rank'] +
        0.3 * customer_metrics['purchase_frequency_rank'] + 
        0.3 * customer_metrics['avg_order_value_rank']
    )
    
    # Segment assignment based on score distribution
    def get_segment(score):
        if score >= 0.8: 
            return 'Premium'
        elif score >= 0.6:
            return 'Loyal'
        elif score >= 0.4:
            return 'Regular'
        elif score >= 0.2:
            return 'Occasional'
        else:
            return 'At Risk'
    
    customer_metrics['segment'] = customer_metrics['value_score'].apply(get_segment)
    return customer_metrics

def analyze_returns(df):
    # Aggregate return metrics by segment
    return_analysis = df.groupby('segment').agg({
        'order_id': 'count',
        'is_returned': 'sum',
        'total_amount': 'sum'
    }).reset_index()
    
    # Calculate key performance indicators
    return_analysis['return_rate'] = return_analysis['is_returned'] / return_analysis['order_id']
    return_analysis['avg_order_value'] = return_analysis['total_amount'] / return_analysis['order_id']
    
    return return_analysis
```

```sql
-- Customer cohort analysis for retention tracking
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
