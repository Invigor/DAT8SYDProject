# Market Segmentation of Shopping Centre Visitors based on WiFi Tracking Data
General Assembly Data Science Project

## Executive Summary

This project seeks to segment shopping centre visitors based on tracking their mobile device via the shopping centre's WiFi network. This is an unsupervised learning problem based on clustering techniques.

## Background

Shopping centres are real estate owners which generate revenue from retailers (tenants) in terms of leases on physical space within the centre. Leases generally comprise a fixed rental over the tenancy plus additional payments if the tenant exceeds an agreed revenue target. 

The focus of the shopping centre marketing team is to attract more shoppers to the centre and encourage them to spend more money with the retailers. This involves [Above The Line](https://www.feedough.com/atl-btl-ttl-marketing/) (ATL) marketing such as TV, print and radio advertising as well as Below The Line (BTL) marketing to individual shoppers via Email Direct Marketing (EDM) campaigns.

Shopping centres are now being [designed and promoted as destinations](http://communicationscollective.com.au/seven-ways-to-create-a-destination-shopping-centre-development/) by incorporating food and beverage precincts, broadening the tenant mix to include services such as gyms, cinemas and medical centres, and ensuring appropriate spaces for social interaction.

Once shoppers are in the centre, they are [encouraged to stay longer](https://www.choice.com.au/shopping/everyday-shopping/shopping-centres/articles/shopping-centre-design) (dwell) and spend more via a number of means including:
- Positioning anchor tenants at different ends of the centre requiring shoppers to walk past smaller retail outlets.
- Providing other services such as children's play areas
- Offering WiFi hotspots to allow shoppers to easily connected to the internet for personal or work activities

One of the main challenges for shopping centre marketing staff is not really understanding who shoppers are and why they may be visiting the centre. Since they don't have a direct transactional relationship with shoppers (compared with retailers), establishing loyalty programs such as FlyBuys or Everyday Rewards is not really feasible. 

In order to target shoppers with EDM campaigns, they rely on shoppers signing up to their mailing list via the shopping centre website or physical cards available from the concierge. This sign up process is only able to elicit the most basic information required for a marketing campaign such as name and email address as well as gender and age range in some cases. 

For shopping centre marketing staff, this information only provides the ability to segment shoppers based on demographics which is not considered particularly useful for targeting and as a result, most campaigns are sent as generic emails to entire subscriber base.

Ideally the campaigns could be customised for specific shopper segments based on the types of product, retailers and services of interest to the shopper. For example, a mother with a young baby may be targeted with a campaign including offers from Baby Bunting and promoting the centre's upgraded mothers room while a single fashion forward female would respond to information about a newly opened day spa promotions and a 2-for-1 coffee voucher. In both cases the demographic profile (gender and age) is identical but their interests and needs are totally different.

## WiFi Shopper Tracking

As indicated above, shopping centres offer free WiFi to shoppers as an encouragement to stay longer and allowing them to do online product research, connect to social networks or stay updated with news and information. The WiFi registration process is also used to opt-in to the shopping centre mailing list including the collection of contact details.

WiFi also provides the [ability to track shoppers](http://bgr.com/2014/02/19/how-smartphone-wi-fi-tracking-works/) including those who have not connected to the network. Smartphones and other WiFi enabled devices emit regular signals called probe requests in an attempt to identify nearby WiFi networks which include the device's unique identifier (MAC address). The shopper can then be located to the access point which detects the probe request with the highest signal strength.

Where multiple access points detect the same probe request, the relative signal strength at each access point can be used to trilaterate the position of the shopper in 2 dimensions to an accuracy of 5m - 8m.

The location information combined with a persistent identifier for the device (and therefore the shopper) allows a number of different metrics to be generated as a proxy for shopping centre visitation activing including:
- Total shopper numbers
- Shopper numbers by zone
- Shopper numbers by day of week and time of day
- Shopper movement within the centre
- Shopper frequency and recency of visit 

## Segmentation Hypotheses

The focus of this project is to segment shoppers based on movement within the centre which is hypothesized to indicate the type of product the shopper is seeking to purchase.

It is also believe that other characteristics of the shopper and their visit may be useful contributors to the segmentation model including demographics (gender and age) as well as whether the shopper is visiting during the week, on Thursday night or on  the weekend.

Two other key segments will be retailer and shopping centre staff as well as static devices including mobile phones on display in consumer electronics retailers and PoS terminals within the outlets. To help segment these two groups, the number of days each device was detected within the period will be included in the analysis. 

## Data Set

The data set comprised shopper sign-up information and movement data collected from the shopping centre WiFi in a Sydney home maker centre between 1 January 2017 and 30 June 2017. The WiFi network includes 10 individual access points deployed as zones which each cover a number of retailers or centre areas. WiFi coverage is prioritised for the common areas i.e. the central walkway, but connection and tracking will continue into most tenancies. The spacing between access points precludes trilateration of the shopper's exact location so the granularity of location is limited to one of 10 zones within the centre.

### Data Extraction Queries
```SQL
-- Retrieve raw subscriber zone dwell data via API
create table scmp_visitor_dwell_ms as 
(
select 
  unnest(subscriber_id) as shopper,
  resolution_ts::date as date,
  zone,
  dwell 
from 
  api_report_history_insights(null::find_static_devices_dwell,'2017-01-01','2017-06-30','2000-01-01','2000-01-01','find static devices dwell','scmp','All','All','All','All','All','All','All','All','where 1=1','All','where 1=1')
);

-- Convert into a cross-tab table based on zone name with combined date and shopper ID
create table scmp_visitor_dwell_crosstab_ms as 
(
SELECT 
    *
FROM 
    crosstab(
        'select 
            date::text || ''#'' || shopper::text as shopper_date,
            zone, 
            sum(extract(epoch from dwell::interval))/60 as dwell
        from 
            scmp_visitor_dwell_ms
        where 
            zone in (''Gallery King Furniture'',''Ground BBQ Ranch'',''Ground Dish Kiosk'',''Ground Freedom'',''Level 1 Dare Gallery'',''Level 1 JB HiFi'',''Level 1 Nick Scali'',''Level 2 Harvey Norman'',''Lower Ground Baby Bunting'',''Lower Ground Escalators'')
        group by 
            shopper,
            date,
            zone 
        order by 
            1,2,3',
            $$select unnest('{Gallery King Furniture,Ground BBQ Ranch,Ground Dish Kiosk,Ground Freedom,Level 1 Dare Gallery,Level 1 JB HiFi,Level 1 Nick Scali,Level 2 Harvey Norman,Lower Ground Baby Bunting,Lower Ground Escalators}'::text[])$$
            )
AS 
    scmp_visitor_dwell_ms(
        shopper_date                text,
        "Gallery King Furniture"    integer, 
        "Ground BBQ Ranch"          integer, 
        "Ground Dish Kiosk"         integer, 
        "Ground Freedom"            integer, 
        "Level 1 Dare Gallery"      integer, 
        "Level 1 JB HiFi"           integer, 
        "Level 1 Nick Scali"        integer, 
        "Level 2 Harvey Norman"     integer, 
        "Lower Ground Baby Bunting" integer, 
        "Lower Ground Escalators"   integer)
);

-- Create table with each shopper's arrival time
create table scmp_visitor_arrival_ms as
(
select
    subscriber_id as shopper,
    date_trunc('day',resolution_ts) as date,
    min(resolution_ts) as arrival
from
    api_report_history_insights(null::VO_DH,'2017-01-01','2017-06-30','2000-01-01','2000-01-01','VO DH','scmp','All','All','All','All','All','All','All','All','where 1=1','All','where 1=1')
group by
    shopper,
    date
);

-- Add extra columns from shopper ID, date, number of visits and day of week type
alter table scmp_visitor_dwell_crosstab_ms add column shopper integer;
alter table scmp_visitor_dwell_crosstab_ms add column date date;
alter table scmp_visitor_dwell_crosstab_ms add column visits integer;
alter table scmp_visitor_dwell_crosstab_ms add column day_of_week_type integer;
alter table scmp_visitor_dwell_crosstab_ms add column arrival integer;

-- Update table to set new columns
update scmp_visitor_dwell_crosstab_ms set shopper = split_part(shopper_date,'#',2)::integer;
update scmp_visitor_dwell_crosstab_ms set date = split_part(shopper_date,'#',1)::date;
update scmp_visitor_dwell_crosstab_ms set arrival = extract(epoch from a.arrival)/3600 from (select shopper,date,arrival from scmp_visitor_arrival_ms) a where scmp_visitor_dwell_crosstab_ms.shopper = a.shopper and scmp_visitor_dwell_crosstab_ms.date = a.date;
update scmp_visitor_dwell_crosstab_ms set visits = a.visits from (select shopper,count(*) as visits from scmp_visitor_arrival_ms group by shopper) a where scmp_visitor_dwell_crosstab_ms.shopper = a.shopper;
update scmp_visitor_dwell_crosstab_ms set day_of_week_type = extract(dow from date);
