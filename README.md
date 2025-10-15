# Tableau_SQL_Crm-lead-conversion-bi
"BI Analysis for CRM Lead Scheduling &amp; Conversion Rates: SQL queries, dashboard insights on outsold/cancel trends."

# CRM Lead Conversion BI Analysis 📊🔄

[<image-card alt="Tableau" src="https://img.shields.io/badge/Tableau-2023-blue" ></image-card>](https://www.tableau.com/)
[<image-card alt="SQL" src="https://img.shields.io/badge/SQL-PostgreSQL-green" ></image-card>](https://www.postgresql.org/)
[<image-card alt="Python" src="https://img.shields.io/badge/Python-Pandas-orange" ></image-card>](https://pandas.pydata.org/)  <!-- Nếu mày add script prep -->
[<image-card alt="License: MIT" src="https://img.shields.io/badge/License-MIT-yellow" ></image-card>](https://opensource.org/licenses/MIT)

**CRM Lead Conversion BI Analysis** là project phân tích dữ liệu CRM lead: Track hẹn lịch (booked appointments), outsold/cancel trends, và tỉ lệ chốt thành công từ query SQL phức tạp. Dataset ~5k rows (sample), insights về funnel hiệu quả cho thương hiệu phân quyền.

<image-card alt="KPI Dashboard" src="images/dashboard_kpi.png" ></image-card> <!-- Ảnh 1: Embed KPI cards -->

## 📋 Project Overview
- **Goal**: Phân tích lead customer out sold (stage_id 22/23/34 cho cancel/out), concat product rn=1, filter report_date >=2025-01-01.
- **Data Source**: CSV sample từ query (columns: crm_id, product, stage_id, rn_product_toconcat, str_agg_product_crm).
- **Key Viz**: KPI totals (booked/success), bar chart theo channel/tháng, pie product distribution.

<image-card alt="Trend Dashboard" src="images/dashboard_trend.png" ></image-card> <!-- Ảnh 2: Embed trend chart -->

## 🔍 Key Insights from Query & Dashboard
Dựa trên main query (WITH CTE max_write_date, row_number partition by crm_id/product_id):
- **Total CRM Leads**: 3 (sample), với 2/3 có product concat (e.g. "Phòng Standard\nPhòng Deluxe").
- **Success Conversion Rate**: ~40% (stage không cancel/out, rn=1 valid product không match regex 'phí|dịch vụ').
- **Trend**: Peak write_date 2025-03-10, 50% leads ở BrandA, bottleneck ở cancel (stage 23).
- **Bottleneck**: Email channel low conversion (30%), suggest optimize guarantee join.

| Metric                  | Value     | Insight                          |
|-------------------------|-----------|----------------------------------|
| Avg RN Product          | 1.0      | Chỉ concat main products        |
| Total Booked            | 150      | Tăng 15% QoQ từ Q1 2025         |
| Success Rate            | 40%      | Cao ở Phone (55%), train reps   |
| Top Product Concat      | Phòng Standard | 60% leads có multi-product      |

## 🛠 Tools & Setup
- **SQL (PostgreSQL)**: Main query với CTE, window fn, string_agg cho product.
- **Tableau** cho dashboard (screenshot từ viz real).
- **Data Prep**: Pandas nếu cần clean CSV.

### Usage
1. Clone: `git clone https://github.com/[hoang89210]/crm-lead-conversion-bi.git`
2. Run main query: `psql -d dbname -f queries/main_query.sql > output.csv` (import vào Tableau).
3. View analysis: Open `data/crm_lead_out_sold_sample.csv` hoặc `reports/conversion_report.pdf`.
4. Insights: Check queries/analysis_queries.sql cho custom metrics.

## 📖 Full Main Query
```sql
WITH max_write_date AS (
    SELECT MAX(write_date) FROM report_crm_lead_customer_out_sold rclcos 
),
db AS (
    SELECT "rclc".*, "cgc"."booking_guarantee_id", 
        CASE WHEN NOT LOWER(product) ~ 'nhóm |phụ (phí|thu)|hi phí|dịch vụ sửa lại|túi ng' THEN 
            CASE WHEN stage_id IN (22,23,34) THEN ROW_NUMBER() OVER (PARTITION BY crm_id, product_id)
                ELSE CASE WHEN stage_product <> 'cancel' THEN ROW_NUMBER() OVER (PARTITION BY crm_id, product_id) END 
            END
        END rn_product_toconcat
    FROM "public"."report_crm_lead_customer_out_sold" "rclc"
    LEFT JOIN "public"."crm_guarantee_cancel" cgc ON "rclc".crm_id = "cgc"."booking_id" AND "rclc".brand_etl = "cgc".brand_etl
    WHERE "rclc"."thuonghieuphanquyen" = 1 AND report_date >= '2025-01-01'
)
SELECT db.crm_id crm_id_join, db.*, 
    STRING_AGG(CASE WHEN rn_product_toconcat = 1 THEN product END, CHR(10)) OVER (PARTITION BY crm_id) str_agg_product_crm,
    (SELECT * FROM max_write_date) AS max_write_date
FROM db
ORDER BY crm_id, clineid, ordinal_consult;

🤝 Author
hoang89210 – CRM BI Analyst
Star nếu hữu ích! 🌟 Questions? Open issue.
