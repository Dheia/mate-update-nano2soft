## Benefits, Advantages, and Disadvantages of the `getPreAggregatedReport` Function

### What is the `getPreAggregatedReport` Function?
It is a specialized function within `ReportsManager` designed to execute registered reports but using **pre‑aggregated tables** instead of the original tables. This feature is activated by adding `'pre_aggregated' => true` in the report definition, specifying the name of the aggregated table via the `aggregate_table` key (otherwise, the system assumes the original table name with `_aggregates` appended).

---

### Main Benefits

1. **Significant Query Performance Improvement**  
   - When dealing with millions of records, performing aggregation operations (SUM, AVG, COUNT) on the original table consumes time and resources. Pre-aggregated tables contain pre-calculated results, making the query execute in fractions of a second.

2. **Reduced Database Load**  
   - Instead of executing complex queries scanning millions of rows each time, only a few rows are read from the aggregated table. This reduces CPU and I/O usage on the database server.

3. **Instant Dashboard Responsiveness**  
   - Reports displaying key performance indicators (KPIs) such as monthly sales totals or average ratings benefit ideally from pre-aggregation, as data appears quickly without delay.

4. **Flexible Periodic Updates**  
   - Aggregated tables can be created and updated via scheduled jobs (Cron Jobs) hourly, daily, or as needed. This allows balancing between data freshness and performance.

5. **Full Compatibility with the Existing Reporting System**  
   - The function follows the same pattern as `runStoredReport`, supporting filters, pagination, and the same output structure, making its use transparent to both developers and end-users.

---

### Technical Advantages

- **Reuse the Same Report Definition** – No need to modify `query_config`; simply add the `pre_aggregated` property to the report definition.
- **Full Filter Support** – Filters (`filterValues`) are integrated with the aggregated table query using the same mechanism, provided the aggregated tables contain the columns used in the conditions.
- **Flexibility in Table Naming** – The aggregated table name can be customized via `aggregate_table` if it differs from the default naming convention.
- **Scalability** – The same concept can be applied to multiple reports without impacting the underlying infrastructure.

---

### Disadvantages and Limitations

1. **Requires Additional Maintenance**  
   - Aggregated tables must be created and updated regularly, adding complexity to development and operations (requiring periodic update scripts).

2. **Data Latency**  
   - If the aggregated table is updated only once a day, reports will not reflect real-time changes. This is unsuitable for applications requiring immediate data.

3. **Increased Storage Consumption**  
   - Storing aggregated copies of data consumes additional disk space, especially if multiple different aggregation dimensions exist.

4. **Loss of Detailed Data**  
   - Pre-aggregation eliminates the ability to access individual row-level data. Therefore, it is not suitable for detailed reports that require displaying each row separately.

5. **Pre-planned Aggregation Dimensions**  
   - Aggregation dimensions (e.g., `GROUP BY month, category`) must be known in advance when designing the aggregated table, reducing flexibility compared to direct queries that can aggregate by any column at runtime.

6. **Complexity with Unforeseen Filters**  
   - If a report includes a filter on a column not present in the aggregated table, the condition may not be executable or the results may be inaccurate.

---

### When to Use `getPreAggregatedReport`?

- **Operational Reports** displaying key performance indicators that require high speed.
- **Dashboards** that refresh every few hours.
- **Historical Reports** that do not require real-time updates.

### When to Avoid Using It?

- **Detailed Reports** that display data row by row.
- **Reports requiring real-time data**.
- **Systems with limited storage capacity**.

---

### Conclusion
The `getPreAggregatedReport` function provides a powerful solution to the performance issue of aggregated reports on large datasets, while maintaining a smooth developer experience. However, it requires careful planning to balance performance, data freshness, and query flexibility. Used wisely, it enables building a reporting system that responds quickly even with millions of records.