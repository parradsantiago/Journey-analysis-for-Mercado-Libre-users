WITH first_visits AS (
    SELECT DISTINCT user_id, country
    FROM mercadolibre_funnel
    WHERE event_name = 'first_visit' 
        AND event_date BETWEEN '2025-01-01' AND '2025-08-31'
        
),

    select_item AS (
    SELECT DISTINCT user_id, country
    FROM mercadolibre_funnel
    WHERE event_name IN ('select_item','select_promotion')
        AND event_date BETWEEN '2025-01-01' AND '2025-08-31'
),
    
    add_to_cart AS (
   SELECT DISTINCT user_id, country
    FROM mercadolibre_funnel
    WHERE event_name = 'add_to_cart'
        AND event_date BETWEEN '2025-01-01' AND '2025-08-31'
),
    
    begin_checkout AS (
    SELECT DISTINCT user_id, country
    FROM mercadolibre_funnel
    WHERE event_name = 'begin_checkout'
        AND event_date BETWEEN '2025-01-01' AND '2025-08-31'
),
    
    add_shipping_info AS (
    SELECT DISTINCT user_id, country
    FROM mercadolibre_funnel
    WHERE event_name = 'add_shipping_info'
        AND event_date BETWEEN '2025-01-01' AND '2025-08-31'
),

    add_payment_info AS (
    SELECT DISTINCT user_id, country
    FROM mercadolibre_funnel
    WHERE event_name = 'add_payment_info'
        AND event_date BETWEEN '2025-01-01' AND '2025-08-31'
),
    purchase AS (
    SELECT DISTINCT user_id, country
    FROM mercadolibre_funnel
    WHERE event_name = 'purchase'
        AND event_date BETWEEN '2025-01-01' AND '2025-08-31'
),
    
 funnel_counts AS (   

SELECT 
    fv.country,

    
COUNT(fv.user_id) AS usuarios_first_visits,
COUNT(si.user_id) AS usuarios_select_item,
COUNT(ac.user_id) AS usuarios_add_to_cart,
COUNT(bc.user_id) AS usuarios_begin_checkout,
COUNT(ash.user_id) AS usuarios_add_shipping_info,
COUNT(api.user_id) AS usuarios_add_payment_info,
COUNT(p.user_id) AS usuarios_purchase


-- Partiendo de FROM first_visit se hacen los LEFT joins con user_ide con cada una de las CTEs creadas anteriormente   

FROM first_visits AS fv

LEFT JOIN select_item AS si ON fv.user_id = si.user_id AND fv.country = si.country

LEFT JOIN add_to_cart AS ac ON fv.user_id  = ac.user_id AND fv.country = ac.country

LEFT JOIN begin_checkout AS bc ON fv.user_id  = bc.user_id AND fv.country = bc.country

LEFT JOIN add_shipping_info AS ash ON fv.user_id  = ash.user_id AND fv.country = ash.country

LEFT JOIN add_payment_info AS api ON fv.user_id  = api.user_id AND fv.country = api.country

LEFT JOIN purchase AS p ON fv.user_id = p.user_id AND fv.country =p.country
    
GROUP BY fv.country)

SELECT
-- Tasas de conversion para cada etapa del funnel partiendo de first_visit
country,
    
    

    
usuarios_select_item * 100.0 / NULLIF(usuarios_first_visits,0)  AS conversion_select_item,
usuarios_add_to_cart * 100.0 / NULLIF(usuarios_first_visits,0) AS conversion_add_to_cart,
usuarios_begin_checkout * 100.0 / NULLIF(usuarios_first_visits,0) AS conversion_begin_checkout,
usuarios_add_shipping_info * 100.0 / NULLIF(usuarios_first_visits,0) AS conversion_add_shipping_info,
usuarios_add_payment_info * 100.0 / NULLIF(usuarios_first_visits,0) AS conversion_add_payment_info,
usuarios_purchase * 100.0 / NULLIF(usuarios_first_visits,0) AS conversion_purchase
    
    
FROM funnel_counts
    
ORDER BY conversion_purchase DESC
