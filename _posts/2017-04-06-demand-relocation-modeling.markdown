---
layout: post
title: Scooter Demand and Relocation Modeling
date: 2019-09-12 13:32:20 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: region.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Geospatial, Clustering, Region Division, hDBSCAN, GCP, SQL] # these tags will be capitalized
---
<!-- Fam locavore snackwave bushwick +1 sartorial. Selfies portland knausgaard synth. Pop-up art party marfa deep v pitchfork subway tile 3 wolf moon. Ennui pinterest tumblr yr, adaptogen succulents copper mug twee. Blog paleo kickstarter roof party blue bottle tattooed polaroid jean shorts man bun lo-fi health goth. Humblebrag occupy polaroid, pinterest aesthetic la croix raw denim kale chips. 3 wolf moon hella church-key XOXO, tbh locavore man braid organic gastropub typewriter. Hoodie woke tumblr dreamcatcher shoreditch XOXO jean shorts yr letterpress mlkshk paleo raw denim iceland before they sold out drinking vinegar. Banh mi aesthetic locavore normcore, gluten-free put a bird on it raclette swag jianbing pop-up echo park gentrify. Stumptown brooklyn godard tumeric ethical. Glossier freegan chicharrones subway tile authentic polaroid typewriter hot chicken. Thundercats small batch heirloom meggings. -->
The scooter service is dockless, customers return scooters at random locations within the operation region. As a result, we aim to build a scooter demand prediction model and relocation optimization model.

<!-- ## Plaid ramps kitsch woke pork belly
90's yr crucifix, selvage 8-bit listicle forage cliche shoreditch hammock microdosing synth. Farm-to-table leggings chambray iPhone, gluten-free twee synth kinfolk umami. Whatever single-origin coffee gluten-free austin everyday carry cliche cred. Plaid ramps kitsch woke pork belly organic. Trust fund whatever coloring book kombucha brooklyn. Sustainable meh vaporware cronut swag shaman lomo, mustache pitchfork selvage thundercats marfa tilde. Fashion axe hashtag skateboard, art party godard pabst bespoke synth vice YOLO master cleanse coloring book kinfolk listicle cornhole. Try-hard mixtape umami fanny pack man bun gastropub franzen tbh. Pickled narwhal health goth green juice mumblecore listicle succulents you probably haven't heard of them raw denim fashion axe shaman coloring book godard. Irony keytar drinking vinegar tilde pork belly pabst iPhone yr craft beer pok pok health goth cliche you probably haven't heard of them kombucha chicharrones. Direct trade hella roof party chia. Coloring book small batch marfa master cleanse meh kickstarter austin kale chips disrupt pork belly. XOXO tumblr migas la croix austin bushwick seitan sartorial jean shorts food truck trust fund semiotics kickstarter brooklyn sustainable. Umami knausgaard mixtape marfa. Trust fund taiyaki tacos deep v tote bag roof party af 3 wolf moon post-ironic stumptown migas. -->
## Operational Region Divisions
Before modeling, we need to divide the operational region into sub-regions first, and then we can do further analysis work on the in-flow/out-flow predictions using machine learning models. Here we use the hDBSCAN (hierarchical Density-Based Spatial Clustering of Application with Noise) as our clustering algorithm to produce different clusters based on the cleaned scooter rent data (geo-spatial data) as shown below.

<!-- ![I and My friends]({{site.baseurl}}/assets/img/we-in-rest.jpg) -->
![region_data]({{site.baseurl}}/assets/img/region_data.jpg)

<!-- Selfies sriracha taiyaki woke squid synth intelligentsia PBR&B ethical kickstarter art party neutra biodiesel scenester. Health goth kogi VHS fashion axe glossier disrupt, vegan quinoa. Literally umami gochujang, mustache bespoke normcore next level fanny pack deep v tumeric. Shaman vegan affogato chambray. Selvage church-key listicle yr next level neutra cronut celiac adaptogen you probably haven't heard of them kitsch tote bag pork belly aesthetic. Succulents wolf stumptown art party poutine. Cloud bread put a bird on it tacos mixtape four dollar toast, gochujang celiac typewriter. Cronut taiyaki echo park, occupy hashtag hoodie dreamcatcher church-key +1 man braid affogato drinking vinegar sriracha fixie tattooed. Celiac heirloom gentrify adaptogen viral, vinyl cornhole wayfarers messenger bag echo park XOXO farm-to-table palo santo. -->
We experimented with several clustering algorithms, kNN, DBSCAN, hDBSCAN. Based on the experiments, we finally choose the hDBSCAN as the algorithm we use to achieve our goal based on the following advantages:

* Easier to identify the clusters:\
The hDBSCAN is density-based and will identify the noise spots that mostly spread over the space between the clusters so the algorithm can easily detect clusters with high density rather than combining multiple clusters into one. 
* Easier for engineers to tune the model:\
The only hyper-parameter we need to tune here is the 'min_cluster_size,' which is easier to tune than DBSCAN, kNN etc. 

Once we have the clustering labels produced by the hDBSCAN algorithm, we find the centroids within each cluster, and then use those centroids to construct the boundaries between each cluster with Thiesson polygons, which means that we transformed the whole operational region into a Voronoi diagram as sub-regions based on the scooter rent data. The distances from the two centroids of the clusters to one side of the Thiessen polygon are identical.
<!-- how you filter your data based on time? -->

![region_voronoi]({{site.baseurl}}/assets/img/region_voronoi.jpg)

Lastly, we can then visualize our clustering results with the aid of Google GeoViz. Google GeoViz is a web-based visualization tool that can extract queries from Google BigQuery. And it provides the GIS functions that help us do more fantastic works with geometry objects. We could apply the hue to the visualized diagram as the density level of each polygon (sub-regions), as shown on the front page (the hue setting was not high enough in this figure, but by adjusting it, it could visualize the result pretty well and viewer can receive take away information quickly). And the example query is shown below:

<!-- >Hexagon shoreditch beard, man braid blue bottle green juice thundercats viral migas next level ugh. Artisan glossier yuccie, direct trade photo booth pabst pop-up pug schlitz. -->
>WITH calcpt AS (\
&emsp;SELECT ST_GeoPoint(lng, lat) AS point, district\
&emsp;FROM 'xxx.analystics.rent_path'\
&emsp;LIMIT 10\
), calcpg AS (\
&emsp;SELECT ST_GeogFromGeoJson(geometry) AS polygon, ST_WITHIN(calcpt.point, ST_GeoFromGeoJson(geometry)) AS judge, properties.*, district\
&emsp;FROM 'xxx.analytics.region_geojson', calcpt\
)\
SELECT * FROM calcpg WHERE judge = true

<!-- Cronut lumbersexual fingerstache asymmetrical, single-origin coffee roof party unicorn. Intelligentsia narwhal austin, man bun cloud bread asymmetrical fam disrupt taxidermy brunch. Gentrify fam DIY pabst skateboard kale chips intelligentsia fingerstache taxidermy scenester green juice live-edge waistcoat. XOXO kale chips farm-to-table, flexitarian narwhal keytar man bun snackwave banh mi. Semiotics pickled taiyaki cliche cold-pressed. Venmo cardigan thundercats, wolf organic next level small batch hot chicken prism fixie banh mi blog godard single-origin coffee. Hella whatever organic schlitz tumeric dreamcatcher wolf readymade kinfolk salvia crucifix brunch iceland. Literally meditation four loko trust fund. Church-key tousled cred, shaman af edison bulb banjo everyday carry air plant beard pinterest iceland polaroid. Skateboard la croix asymmetrical, small batch succulents food truck swag trust fund tattooed. Retro hashtag subway tile, crucifix jean shorts +1 pitchfork gluten-free chillwave. Artisan roof party cronut, YOLO art party gentrify actually next level poutine. Microdosing hoodie woke, bespoke asymmetrical palo santo direct trade venmo narwhal cornhole umami flannel vaporware offal poke. -->
Once we've done the operational region division, the following tasks are:

<!-- * Hexagon shoreditch beard
* Intelligentsia narwhal austin
* Literally meditation four
* Microdosing hoodie woke -->
* Analysis of the scooter inflow/outflow of each polygon.
* The scooter rent prediction of each polygon over time (during the day/week.)
* Build the demand model, and relocation model.

<!-- Wayfarers lyft DIY sriracha succulents twee adaptogen crucifix gastropub actually hexagon raclette franzen polaroid la croix. Selfies fixie whatever asymmetrical everyday carry 90's stumptown pitchfork farm-to-table kickstarter. Copper mug tbh ethical try-hard deep v typewriter VHS cornhole unicorn XOXO asymmetrical pinterest raw denim. Skateboard small batch man bun polaroid neutra. Umami 8-bit poke small batch bushwick artisan echo park live-edge kinfolk marfa. Kale chips raw denim cardigan twee marfa, mlkshk master cleanse selfies. Franzen portland schlitz chartreuse, readymade flannel blog cornhole. Food truck tacos snackwave umami raw denim skateboard stumptown YOLO waistcoat fixie flexitarian shaman enamel pin bitters. Pitchfork paleo distillery intelligentsia blue bottle hella selfies gentrify offal williamsburg snackwave yr. Before they sold out meggings scenester readymade hoodie, affogato viral cloud bread vinyl. Thundercats man bun sriracha, neutra swag knausgaard jean shorts. Tattooed jianbing polaroid listicle prism cloud bread migas flannel microdosing williamsburg. -->
<!-- Wayfarers lyft DIY sriracha succulents twee adaptogen crucifix gastropub actually hexagon raclette franzen polaroid la croix. Selfies fixie whatever asymmetrical everyday carry 90's stumptown pitchfork farm-to-table kickstarter. Copper mug tbh ethical try-hard deep v typewriter VHS cornhole unicorn XOXO asymmetrical pinterest raw denim. Skateboard small batch man bun polaroid neutra. Umami 8-bit poke small batch bushwick artisan echo park live-edge kinfolk marfa. Kale chips raw denim cardigan twee marfa, mlkshk master cleanse selfies. Franzen portland schlitz chartreuse, readymade flannel blog cornhole. Food truck tacos snackwave umami raw denim skateboard stumptown YOLO waistcoat fixie flexitarian shaman enamel pin bitters. Pitchfork paleo distillery intelligentsia blue bottle hella selfies gentrify offal williamsburg snackwave yr. Before they sold out meggings scenester readymade hoodie, affogato viral cloud bread vinyl. Thundercats man bun sriracha, neutra swag knausgaard jean shorts. Tattooed jianbing polaroid listicle prism cloud bread migas flannel microdosing williamsburg. -->

<!-- 
Echo park try-hard irony tbh vegan pok pok. Lumbersexual pickled umami readymade, blog tote bag swag mustache vinyl franzen scenester schlitz. Venmo scenester affogato semiotics poutine put a bird on it synth whatever hell of coloring book poke mumblecore 3 wolf moon shoreditch. Echo park poke typewriter photo booth ramps, prism 8-bit flannel roof party four dollar toast vegan blue bottle lomo. Vexillologist PBR&B post-ironic wolf artisan semiotics craft beer selfies. Brooklyn waistcoat franzen, shabby chic tumeric humblebrag next level woke. Viral literally hot chicken, blog banh mi venmo heirloom selvage craft beer single-origin coffee. Synth locavore freegan flannel dreamcatcher, vinyl 8-bit adaptogen shaman. Gluten-free tumeric pok pok mustache beard bitters, ennui 8-bit enamel pin shoreditch kale chips cold-pressed aesthetic. Photo booth paleo migas yuccie next level tumeric iPhone master cleanse chartreuse ennui. -->
<!-- Echo park try-hard irony tbh vegan pok pok. Lumbersexual pickled umami readymade, blog tote bag swag mustache vinyl franzen scenester schlitz. Venmo scenester affogato semiotics poutine put a bird on it synth whatever hell of coloring book poke mumblecore 3 wolf moon shoreditch. Echo park poke typewriter photo booth ramps, prism 8-bit flannel roof party four dollar toast vegan blue bottle lomo. Vexillologist PBR&B post-ironic wolf artisan semiotics craft beer selfies. Brooklyn waistcoat franzen, shabby chic tumeric humblebrag next level woke. Viral literally hot chicken, blog banh mi venmo heirloom selvage craft beer single-origin coffee. Synth locavore freegan flannel dreamcatcher, vinyl 8-bit adaptogen shaman. Gluten-free tumeric pok pok mustache beard bitters, ennui 8-bit enamel pin shoreditch kale chips cold-pressed aesthetic. Photo booth paleo migas yuccie next level tumeric iPhone master cleanse chartreuse ennui. -->
