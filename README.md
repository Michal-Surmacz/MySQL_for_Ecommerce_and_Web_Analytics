# MySQL_for_Ecommerce_and_Web_Analytics

The main goal of the project was to reflect real, real working conditions at position Ecommerce and Web Analytics.

# Objective „MySQL_for_Ecommerce_and_Web_Analytics”:
1. Discuss eCommerce databases, download Community Server and Workbench, and create the project database,
2. Identify top traffic sources, measure their conversion rates, analyze trends, and use segmentation for bidding optimization,
3. Find the most-visited pages and top entry pages, calculate bounce rates, build conversion funnels, and analyze tests
4. Compare marketing channels, understand relative performance, optimize a channel portfolio, and analyze trends,
5. Analyze sales, build product-level conversion funnels, learn about cross-selling, and measure the impact of launching new products,
6. Learn about behaviors of repeat visitors and purchasers, and compare new and repeat visitor website conversion patterns. 


# My objectives in the final project included:
1. Tell the story of your company's growth, using trended performance data,
2. Use the database to explain how you’ve been able to produce growth, by diving in to channels and website optimizations,
3. Flex my analytical muscles so the VCs know your company is a serious data-driven shop

# Solution:
```
USE mavenfuzzyfactory;

/*
1. First, I’d like to show our volume growth. Can you pull overall session and order volume, trended by quarter for the life of the business? Since the most recent quarter is incomplete, you can decide how to handle it.
*/

SELECT 
    YEAR(website_sessions.created_at) AS year,
    QUARTER(website_sessions.created_at) AS quarter,
    COUNT(DISTINCT website_sessions.website_session_id) AS session,
    COUNT(DISTINCT orders.order_id) AS orders,
    COUNT(DISTINCT orders.order_id) / NULLIF(COUNT(DISTINCT website_sessions.website_session_id), 0) AS session_to_order_cov_rt
FROM website_sessions
LEFT JOIN orders ON orders.website_session_id = website_sessions.website_session_id
GROUP BY 1, 2
ORDER BY 1, 2;

/*
2. Next, let’s showcase all of our efficiency improvements. I would love to show quarterly figures since we
launched, for session-to-order conversion rate, revenue per order, and revenue per session.
*/

SELECT 
    YEAR(website_sessions.created_at) AS Year,
    QUARTER(website_sessions.created_at) AS Quarter,
    COUNT(DISTINCT orders.order_id) / NULLIF(COUNT(DISTINCT website_sessions.website_session_id), 0) AS SessionToOrderConversionRate,
    SUM(orders.price_usd) / NULLIF(COUNT(DISTINCT orders.order_id), 0) AS RevenuePerOrderUSD,
    SUM(orders.price_usd) / NULLIF(COUNT(DISTINCT website_sessions.website_session_id), 0) AS RevenuePerSessionUSD
FROM 
    website_sessions
LEFT JOIN 
    orders ON orders.website_session_id = website_sessions.website_session_id
GROUP BY 
    1, 2
ORDER BY 
    1, 2;

/*
3. I’d like to show how we’ve grown specific channels. Could you pull a quarterly view of orders from Gsearch
nonbrand, Bsearch nonbrand, brand search overall, organic search, and direct type-in?
*/

SELECT
    YEAR(ws.created_at) AS Year,
    QUARTER(ws.created_at) AS Quarter,
    COUNT(DISTINCT CASE WHEN ws.utm_source = 'gsearch' AND ws.utm_campaign = 'nonbrand' THEN o.order_id ELSE NULL END) AS gsearch_paid_sessions,
    COUNT(DISTINCT CASE WHEN ws.utm_source = 'bsearch' AND ws.utm_campaign = 'nonbrand' THEN o.order_id ELSE NULL END) AS bsearch_paid_sessions,
    COUNT(DISTINCT CASE WHEN ws.utm_campaign = 'brand' THEN o.order_id ELSE NULL END) AS overall_brand_sessions,
    COUNT(DISTINCT CASE WHEN ws.utm_source IS NULL AND http_referer IS NOT NULL THEN o.order_id ELSE NULL END) AS organic_search_sessions,
    COUNT(DISTINCT CASE WHEN ws.utm_source IS NULL AND http_referer IS NULL THEN o.order_id ELSE NULL END) AS direct_type_sessions
FROM website_sessions ws
LEFT JOIN orders o ON o.website_session_id = ws.website_session_id
GROUP BY 1, 2;

/*
4. Next, let’s show the overall session-to-order conversion rate trends for those same channels, by quarter. Please also make a note of any periods where we made major improvements or optimizations.
*/

SELECT
    YEAR(ws.created_at) AS Year,
    QUARTER(ws.created_at) AS Quarter,
    COUNT(DISTINCT CASE WHEN ws.utm_source = 'gsearch' AND ws.utm_campaign = 'nonbrand' THEN o.order_id ELSE NULL END) 
		/ COUNT(DISTINCT CASE WHEN ws.utm_source = 'gsearch' AND ws.utm_campaign = 'nonbrand' THEN ws.website_session_id ELSE NULL END) AS gsearch_nonbrand_cov_rt,
    COUNT(DISTINCT CASE WHEN ws.utm_source = 'bsearch' AND ws.utm_campaign = 'nonbrand' THEN o.order_id ELSE NULL END)
		/ COUNT(DISTINCT CASE WHEN ws.utm_source = 'bsearch' AND ws.utm_campaign = 'nonbrand' THEN ws.website_session_id ELSE NULL END) AS bsearch_nonbrand_cov_rt,
    COUNT(DISTINCT CASE WHEN ws.utm_campaign = 'brand' THEN o.order_id ELSE NULL END) 
		/ COUNT(DISTINCT CASE WHEN ws.utm_campaign = 'brand' THEN ws.website_session_id ELSE NULL END) AS brad_search_cov_rt,
    COUNT(DISTINCT CASE WHEN ws.utm_source IS NULL AND http_referer IS NOT NULL THEN o.order_id ELSE NULL END) 
		/ COUNT(DISTINCT CASE WHEN ws.utm_source IS NULL AND http_referer IS NOT NULL THEN ws.website_session_id ELSE NULL END)  AS organic_search_cov_rt,
    COUNT(DISTINCT CASE WHEN ws.utm_source IS NULL AND http_referer IS NULL THEN o.order_id ELSE NULL END) 
		/ COUNT(DISTINCT CASE WHEN ws.utm_source IS NULL AND http_referer IS NULL THEN ws.website_session_id ELSE NULL END) AS direct_type_cov_rt
FROM website_sessions ws
LEFT JOIN orders o ON o.website_session_id = ws.website_session_id
GROUP BY 1, 2;

/*
5. We’ve come a long way since the days of selling a single product. Let’s pull monthly trending for revenue and margin by product, along with total sales and revenue. Note anything you notice about seasonality.
*/

SELECT
    YEAR(created_at) AS Year,
    MONTH(created_at) AS Month,
    SUM(CASE WHEN product_id = 1 THEN price_usd ELSE NULL END) AS mrfuzzy_rev,
    SUM(CASE WHEN product_id = 1 THEN price_usd - cogs_usd ELSE NULL END) AS mrfuzzy_marg,
    SUM(CASE WHEN product_id = 2 THEN price_usd ELSE NULL END) AS lovebear_rev,
    SUM(CASE WHEN product_id = 2 THEN price_usd - cogs_usd ELSE NULL END) AS lovebear_marg,
    SUM(CASE WHEN product_id = 3 THEN price_usd ELSE NULL END) AS birthdaybear_rev,
    SUM(CASE WHEN product_id = 3 THEN price_usd - cogs_usd ELSE NULL END) AS birthdaybear_marg,
    SUM(CASE WHEN product_id = 4 THEN price_usd ELSE NULL END) AS minibear_rev,
    SUM(CASE WHEN product_id = 4 THEN price_usd - cogs_usd ELSE NULL END) AS minibear_marg,
    SUM(price_usd) AS total_revenue,
    SUM(price_usd - cogs_usd) AS total_marg
FROM order_items
GROUP BY 1, 2
ORDER BY 1, 2;

/*
6. Let’s dive deeper into the impact of introducing new products. Please pull monthly sessions to the /products page, and show how the % of those sessions clicking through another page has changed over time, 
along with a view of how conversion from /products to placing an order has improved.
*/

CREATE TEMPORARY TABLE product_pageviews
SELECT
    website_session_id,
    website_pageview_id,
    created_at AS saw_product_page_at
FROM website_pageviews
WHERE pageview_url ='/products';

SELECT * FROM product_pageviews;

SELECT
    YEAR(saw_product_page_at) AS Year,
    MONTH(saw_product_page_at) AS Month,
    COUNT(DISTINCT product_pageviews.website_session_id) AS session_to_product_page,
    COUNT(DISTINCT website_pageviews.website_session_id) AS session_to_next_page,
    COUNT(DISTINCT website_pageviews.website_session_id) / COUNT(DISTINCT product_pageviews.website_session_id) AS clicking_through_another_page,
    COUNT(DISTINCT orders.order_id) AS orders,
    COUNT(DISTINCT orders.order_id) / COUNT(DISTINCT product_pageviews.website_session_id) AS products_to_order_rt
FROM product_pageviews
LEFT JOIN website_pageviews
ON website_pageviews.website_session_id = product_pageviews.website_session_id
AND website_pageviews.website_pageview_id > product_pageviews.website_pageview_id
LEFT JOIN orders
ON orders.website_session_id = product_pageviews.website_session_id
GROUP BY 1,2;

/* 
7. We made our 4th product available as a primary product on December 05, 2014 (it was previously only a cross-sell
item). Could you please pull sales data since then, and show how well each product cross-sells from one another?
*/

CREATE TEMPORARY TABLE primary_products
SELECT
	order_id,
    primary_product_id,
    created_at AS ordered_at
FROM orders
WHERE created_at > '2014-12-05';

SELECT
	primary_product_id,
    COUNT(DISTINCT order_id) AS total_orders,
    COUNT(DISTINCT CASE WHEN cross_sell_product_id = 1 THEN order_id ELSE NULL END) AS _xsold_p1,
    COUNT(DISTINCT CASE WHEN cross_sell_product_id = 2 THEN order_id ELSE NULL END) AS _xsold_p2,
    COUNT(DISTINCT CASE WHEN cross_sell_product_id = 3 THEN order_id ELSE NULL END) AS _xsold_p3,
    COUNT(DISTINCT CASE WHEN cross_sell_product_id = 4 THEN order_id ELSE NULL END) AS _xsold_p4,
    COUNT(DISTINCT CASE WHEN cross_sell_product_id = 1 THEN order_id ELSE NULL END) / COUNT(DISTINCT order_id) AS p1_xsell_rt,
    COUNT(DISTINCT CASE WHEN cross_sell_product_id = 2 THEN order_id ELSE NULL END) / COUNT(DISTINCT order_id) AS p2_xsell_rt,
    COUNT(DISTINCT CASE WHEN cross_sell_product_id = 3 THEN order_id ELSE NULL END) / COUNT(DISTINCT order_id) AS p3_xsell_rt,
    COUNT(DISTINCT CASE WHEN cross_sell_product_id = 4 THEN order_id ELSE NULL END) / COUNT(DISTINCT order_id) AS p4_xsell_rt
    FROM
(
SELECT
	primary_products.*,
    order_items.product_id AS cross_sell_product_id
FROM primary_products
LEFT JOIN order_items
ON order_items.order_id = primary_products.order_id
AND order_items.is_primary_item = 0
) AS primary_w_cross_sell
GROUP BY 1;
```

Based on MySQL for Ecommerce & Web Analytics coure, created by: Maven Analytics, John Pauler

