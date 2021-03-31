# Creative Automation - General overview


## Intro

In technical terms a creative is defined by certain parameters (channel, format, size, layout) and content (image, video, text, url...).
The parameters are grouped together in a template, content is then linked to the template and the output is a creative.

<img width="639" alt="Screen Shot 2021-03-30 at 11 40 03" src="https://user-images.githubusercontent.com/14069474/112968559-b713c400-914c-11eb-9c6c-061e36362a37.png">


So far everything is amazing. So what is the problem that we are solving? In reality advertising campaigns require incredible amount of creatives. Lets use an example to have a better visual representation.

#### Example

Lets say a company is launching a new product line and they want to acompany it with an advertising campaign. You are in charge of creative production for the entire online part of the advertising campaign.

**Here are their requirements:**
* The company are launching 6 new products and you are required to feature each of them in creative.
* The campaign should be present on the following channels: YouTube (2 formats), Facebook (2 formats) & Twitter (6 formats).
* The campaign needs to be translated to 10 different markets. 

Lets just calculate how manny different creatives are needed:
Each format needs atleast 6 different creatives, one for each product. Then you need to multiply this by 10 for each of the different languages. 

```
  products = 6
  YouTube = 2
  Facebook = 2
  Twitter = 6
  languages = 10
  
  (products * youTube) * languages + (products * Facebook) * languages + (products * Twitter) * languages = 600
```

When you start adding even more languages, products and formats, you can see that the end number is growing exponentialy.
But what if you could only build 10 creatives instead of the original 600?

## Creative automation

You can achieve this by using our creative avtomation plaform aka Celtra CA. Celtra CA tool enables users to scale their creative production by combining templates and content stored in feeds to output different combination of products, formats, sizes, layouts and content. 


<img width="975" alt="Screen Shot 2021-03-30 at 16 45 08" src="https://user-images.githubusercontent.com/14069474/113008483-5d74bf00-9177-11eb-95a2-630b8c912a25.png">


The template contains placeholders (text placeholder, image placeholder, video placeholder, ...), the data is stored in content feeds. When you combine these two data sources the placeholders are replaced with actual data from the feed, each line in the feed represents a creative variant. The output is multiple creative variants (number_of_formats * number_of_lines_in_feed).


Additional benefit of this approach is the ability to update creatives. If for some reason you want to change something in your layout, or you want to replace some content, you just need to do this once. 

For changing the layout, you can do this by updating your template. We will detect the change and give user an option to update also their generated creative variants. This kind of update will result in all existing creatives beeing deleted and recreated for the new tempalte. 

For updating the content, you just need to replace it in the feed editor. Again the change will be detected and after updating, only those creatives that were using the assets that you changed will be deleted and regenerated (if you modify 5 lines in the feed, 5 creatives will be deleted and 5 new will be gewnerated after updating).

### Building a template
If you want to find out more about how to build a template [visit](https://support.celtra.io/essentials/build-a-template).
And [link to martins article]()


### Building a content feed
If you want to find out more about how to build a content feed [visit](https://support.celtra.io/feeds/content-feeds).
And [link to martins article]()


### Distributiong and exporting

After you are happy with how your creatives look there are currently two supported ways to distribute them to your adveritising channels. 

You can directly "push" them to some of the supported third party platforms.

<img width="1415" alt="Screen Shot 2021-03-31 at 10 13 59" src="https://user-images.githubusercontent.com/14069474/113112777-dcb2d300-9209-11eb-8976-cb2e9d6ad82b.png">

Or you can export them locally and manually distribut them to advertising channels of your choice.

For more info about Distributing and Exporting [visit](https://github.com/celtra/mab/wiki/QA-DOC:-Exporting-and-Distribution).



## API
You can use direct api request to get raw data whitout the need to load all the assets. Sometimes this can be helpfull when checking the integrity of the data stored or debuging potential issues.

We will cover only a couple of examples here but a usefull place to look for API endpoints is the [Controler folder](https://github.com/celtra/uab/tree/master/server/library/Celtra/Controllers).


### Creative controler

This is the default call to [Creative controler](https://github.com/celtra/uab/blob/master/server/library/Celtra/Controllers/CreativeController.php).
This call will return all of the data specified in the getDefaultFields() function.
```
https://hub.celtra.io/api/creatives/<creative-id>
```
But you can also specify individual fileds if you are interested in something more specific

```
https://hub.celtra.io/api/creatives/<creative-id>?fields=<creative-controler-endpoint>
```
e.g.

To return only the name of the creative:
```
https://hub.celtra.io/api/creatives/f2c79c04?fields=name
```


To return the creative id and the folder id:
```
https://hub.celtra.io/api/creatives/f2c79c04?fields=id,folderId
```

### Custom Feed controler

[Custom feed controler](https://github.com/celtra/uab/blob/master/server/library/Celtra/Controllers/CustomFeedController.php) is the place to look if you need specific information aboout custom feeds.

```
https://hub.celtra.io/api/customFeeds/<feed-id>
```

To return only the name of the feed:
```
https://hub.celtra.io/api/customFeeds/02e6e586?fields=name
```


To return the feed id and the creation timestamp:
```
https://hub.celtra.io/api/customFeeds/02e6e586?fields=id,creationTimestamp
```
