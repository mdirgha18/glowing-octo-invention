# glowing-octo-invention
SQL Questions
1. Retrieve Restaurant Data from Specific Cities
Find all restaurant listings in Mumbai and Bangalore that have at least 20 reviews and an average rating of 4.8 or higher.
Tables:
restaurants: restaurant_id (integer), name (string), city (string), reviews_count (integer)
reviews: restaurant_id (integer), review_id (integer), stars (integer), submit_date (date)
2. Find Average Orders Per Customer in Each City
Write a query to calculate the average number of orders placed per customer in each city.
Tables:
orders: order_id (integer), customer_id (integer), restaurant_id (integer), order_date (date)
restaurants: restaurant_id (integer), city (string)
3. Click-Through Rate for Restaurant Views and Orders
Calculate the click-through conversion rate (CTR) by dividing the number of orders by the number of restaurant views.
Tables:
restaurant_views: view_id (integer), user_id (integer), view_date (date), restaurant_id (integer)
orders: order_id (integer), user_id (integer), order_date (date), restaurant_id (integer)
4. Average Vacant Days in 2023 for Active Restaurants
Write a query to calculate the average number of vacant days in 2023 for restaurants that are currently active.
Tables:
reservations: restaurant_id (integer), checkin_date (date), checkout_date (date)
restaurants: restaurant_id (integer), is_active (integer)
Power BI Questions
1. Create a Dynamic Visual
Create a dynamic bar chart to display the total orders by city, which updates based on slicers for date and restaurant type.
2. Write a DAX Measure
Write a DAX measure to calculate the week-over-week growth in total orders.
3. Calculated Columns vs. Measures
Explain the key differences between calculated columns and measures in Power BI, providing real-world use cases.
4. Implement Row-Level Security (RLS)
Explain how to implement RLS for restaurant owners so that each owner can only see their restaurant's data.
5. Cumulative Revenue Calculation
Write a DAX formula to calculate the cumulative revenue for restaurants based on order dates.
6. KPI for Occupancy Rates
Create a KPI visual to show restaurant occupancy rates by quarter, and highlight underperforming restaurants.
7. Custom Fiscal Year Date Table
Create a custom date table in Power BI for fiscal years starting in July.
8. Top Restaurants by Revenue
Write a DAX formula to identify the top 10 restaurants by revenue.
9. Repeat Customer Analysis
Calculate the total number of orders made by repeat customers using DAX.
10. Data Cleaning in Power Query
Use Power Query to clean and transform a dataset containing customer order details, removing duplicates and standardizing restaurant names.

### **SQL Questions**

#### **1. Retrieve Restaurant Data from Specific Cities**
**Query:**
```sql
SELECT r.name, r.city, r.reviews_count, AVG(rv.stars) AS avg_rating
FROM restaurants r
JOIN reviews rv ON r.restaurant_id = rv.restaurant_id
WHERE r.city IN ('Mumbai', 'Bangalore')
GROUP BY r.restaurant_id, r.name, r.city, r.reviews_count
HAVING COUNT(rv.review_id) >= 20 AND AVG(rv.stars) >= 4.8;
```

---

#### **2. Find Average Orders Per Customer in Each City**
**Query:**
```sql
SELECT r.city, AVG(order_count) AS avg_orders_per_customer
FROM (
    SELECT o.customer_id, r.city, COUNT(o.order_id) AS order_count
    FROM orders o
    JOIN restaurants r ON o.restaurant_id = r.restaurant_id
    GROUP BY o.customer_id, r.city
) subquery
GROUP BY city;
```

---

#### **3. Click-Through Rate for Restaurant Views and Orders**
**Query:**
```sql
SELECT rv.restaurant_id, 
       COUNT(DISTINCT o.order_id) * 1.0 / COUNT(DISTINCT rv.view_id) AS click_through_rate
FROM restaurant_views rv
LEFT JOIN orders o 
ON rv.restaurant_id = o.restaurant_id AND rv.user_id = o.user_id
GROUP BY rv.restaurant_id;
```

---

#### **4. Average Vacant Days in 2023 for Active Restaurants**
**Query:**
```sql
SELECT r.restaurant_id, 
       AVG(DATEDIFF(COALESCE(MIN(rsv.checkin_date), '2023-12-31'), 
                    COALESCE(MAX(rsv.checkout_date), '2023-01-01'))) AS avg_vacant_days
FROM restaurants r
LEFT JOIN reservations rsv 
ON r.restaurant_id = rsv.restaurant_id
WHERE r.is_active = 1 AND YEAR(rsv.checkin_date) = 2023
GROUP BY r.restaurant_id;
```

---

### **Power BI Questions**

#### **1. Create a Dynamic Visual**
**Steps:**
1. Create slicers for `Date` and `Restaurant Type` in Power BI.
2. Drag `City` to the Axis and `Order Count` (from orders table) to the Values.
3. Apply slicer filters so the bar chart updates dynamically.

---

#### **2. Write a DAX Measure**
**DAX Formula:**
```DAX
WeekOverWeekGrowth = 
VAR CurrentWeek = MAX('Orders'[WeekNumber])
VAR PreviousWeek = CurrentWeek - 1
RETURN
    (CALCULATE(SUM('Orders'[OrderCount]), 'Orders'[WeekNumber] = CurrentWeek) - 
     CALCULATE(SUM('Orders'[OrderCount]), 'Orders'[WeekNumber] = PreviousWeek)) /
    CALCULATE(SUM('Orders'[OrderCount]), 'Orders'[WeekNumber] = PreviousWeek)
```

---

#### **3. Calculated Columns vs. Measures**
**Key Differences:**
- **Calculated Columns**:
  - Computed row-by-row during data load or refresh.
  - Useful for row-level calculations like "Age = Today’s Date - Birth Date."
  - Stored in the model and increases model size.
  
- **Measures**:
  - Computed dynamically during query execution.
  - Useful for aggregate calculations like "Total Sales" or "Average Orders."
  - Does not increase model size.
  
**Real-World Use Case**: 
Use a calculated column for categorizing orders (e.g., "Small" or "Large"). Use a measure to calculate total revenue dynamically.

---

#### **4. Implement Row-Level Security (RLS)**
**Steps:**
1. Create a new role in Power BI Desktop (e.g., "RestaurantOwner").
2. Apply the DAX filter:
   ```DAX
   [OwnerID] = USERPRINCIPALNAME()
   ```
3. Publish to the Power BI Service and assign users to the role in Security Settings.

---

#### **5. Cumulative Revenue Calculation**
**DAX Formula:**
```DAX
CumulativeRevenue = 
CALCULATE(
    SUM('Orders'[Revenue]),
    FILTER(
        ALL('Orders'),
        'Orders'[OrderDate] <= MAX('Orders'[OrderDate])
    )
)
```

---

#### **6. KPI for Occupancy Rates**
**Steps:**
1. Create a measure for occupancy rate:
   ```DAX
   OccupancyRate = SUM('Reservations'[OccupiedSeats]) / SUM('Restaurants'[TotalSeats])
   ```
2. Add a KPI visual, setting the Occupancy Rate as the value, and color-code based on thresholds.

---

#### **7. Custom Fiscal Year Date Table**
**Steps:**
1. Create a new date table using DAX:
   ```DAX
   DateTable = 
   ADDCOLUMNS(
       CALENDAR(DATE(2024,7,1), DATE(2025,6,30)),
       "FiscalYear", IF(MONTH([Date]) >= 7, YEAR([Date]), YEAR([Date]) - 1),
       "FiscalQuarter", "Q" & ROUNDUP(MONTH([Date]) / 3, 0)
   )
   ```

---

#### **8. Top Restaurants by Revenue**
**DAX Formula:**
```DAX
Top10Restaurants = 
TOPN(
    10,
    SUMMARIZE(
        'Orders',
        'Orders'[RestaurantID],
        "TotalRevenue", SUM('Orders'[Revenue])
    ),
    [TotalRevenue], DESC
)
```

---

#### **9. Repeat Customer Analysis**
**DAX Formula:**
```DAX
RepeatCustomers = 
COUNTROWS(
    FILTER(
        SUMMARIZE(
            Orders,
            Orders[CustomerID],
            "OrderCount", COUNT(Orders[OrderID])
        ),
        [OrderCount] > 1
    )
)
```

---

#### **10. Data Cleaning in Power Query**
**Steps:**
1. Remove duplicates using the `Remove Duplicates` option in the Transform tab.
2. Use `Replace Values` to standardize restaurant names.
3. Split columns, trim whitespace, and handle missing values with `Replace Errors`.
