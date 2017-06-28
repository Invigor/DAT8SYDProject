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

## Data Set

The data set comprised shopper sign-up information and movement data collected from the shopping centre WiFi in a Sydney home maker centre. The WiFi network includes 10 individual access points deployed as zones which each cover a number of retailers or centre areas. WiFi coverage is prioritised for the common areas i.e. the central walkway, but connection and tracking will continue into most tenancies. The spacing between access points precludes trilateration of the shopper's exact location so the granularity of location is limited to one of 10 zones within the centre.
