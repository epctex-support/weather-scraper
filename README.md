[https://apify.com/epctex/weather-scraper](https://apify.com/epctex/weather-scraper?fpr=yhdrb)

# Weather Scraper

Weather Scraper is an [Apify actor](https://apify.com/actors) for extracting weather information from [Weather.com](https://weather.com). It gives you access to various available weather data in structured form. It is build on top of [Apify SDK](https://sdk.apify.com/) and you can run it both on [Apify platform](https://my.apify.com) and locally.

- [Input](#input)
- [Output](#output)
- [Compute units consumption](#compute-units-consumption)
- [Extend output function](#extend-output-function)

## Bugs, fixes, updates, and changelog

This scraper is under active development. If you have any feature requests you can create an issue from [here](https://github.com/epctex/weather-scraper/issues).

## Input Parameters

| Field | Type | Description | Default value
| ----- | ---- | ----------- | -------------|
| startUrls | array | List of place urls to be processed | `[]` |
| units | string | Unit system to use for the results | `metric` |
| timeFrame | string | Future time frame you want to extract data for | `today` |
| maxItems | number | Maximum number of actor pages that will be scraped | all found |
| locations | array | List of cities / addresses to be processed | `[]` |
| locationIds | array | List of location ids to be processed | `[]` |
| extendOutputFunction | string | Function that takes a JQuery handle ($) as argument and returns data that will be merged with the default output. More information in [Extend output function](#extend-output-function) | |
| customMapFunction | string | Function that takes each of the output rows as argument and returns data that will be mapped, changed formatting, of the each rows. More information in [Custom map function](#custom-map-function) | |
| proxyConfiguration | object | Proxy settings of the run. If you have access to Apify proxy, leave the default settings. If not, you can set `{ "useApifyProxy": false" }` to disable proxy usage | `{ "useApifyProxy": true }`|

### Input example

```json
{
  "startUrls": [
    {"url":"https://weather.com/weather/today/l/5aea1d50a6d6b9e99cf89ba79f463d67dcf21ea5061990aae1ffc1c7fa8911a9"},
    {"url":"https://weather.com/weather/today/l/ada030b2de8db495da2d93d5a2ecf30de1ce8b54cb09725d19c803543685646d"}
  ],
  "locationIds": [
    "e60256f3426acd3c1b3380921210fcffeab628a120c41d1de03b59a0f0dd32ad",
    "bf217d537cc1c8074ec195ce07fb74de3c1593caa6033b7c3be4645ccc5b01de"
  ],
  "locations": [
    "Montgomery, Alabama",
    "Juneau, Alaska"
  ],
  "timeFrame": "ten_day",
  "units": "metric",
  "maxItems": 10,
  "proxyConfiguration": {
    "useApifyProxy": true
  },
  "extendOutputFunction": "($) => {}",
  "customMapFunction": "(object) => {return {...object, }}"
}
```

### Determining locations

In advanced input section, you can provide `locations` parameter - list of addresses you want to scrape. The actual scraped place is the first result in search box on weather.com. In order to get relevant results (which is not always guaranteed in this case), try to input both city and country name (eg. `Paris, France`, `Vienna, Austria`).

If specific locations are needed, you can provide `locationIds` parameter. Location id is used by weather.com to identify specific location. It can be found in the url of a place, it is the last parameter in path. Few examples:

For New York, NY, which url is `https://weather.com/cs-CZ/weather/tenday/l/f892433d7660da170347398eb8e3d722d8d362fe7dd15af16ce88324e1b96e70` the location id is `f892433d7660da170347398eb8e3d722d8d362fe7dd15af16ce88324e1b96e70`. For London, England with url `https://weather.com/en-UK/weather/today/l/7517a52d4d1815e639ae1001edb8c5fda2264ea579095b0f28f55c059599e074` the location id is `7517a52d4d1815e639ae1001edb8c5fda2264ea579095b0f28f55c059599e074`.

## Compute Unit Consumption

Compute unit consumption was measured on a dataset of 50 US capitals.

When the search was done using `locations` parameter (given as `City, State`), the average compute units consumption was 0.02682.

For the same locations, but given in form of `startUrls`, the average consumption was 0.0152.

And finally, when using `locationIds`, the average consumption was 0.01916.

Keeping in mind that the dataset is small, the conclusion is to prefer `startUrls` or `locationIds` over `locations`, if you need to scrape larger amounts of locations. This is due to the fact that the search for location id is omitted in the first two cases.

## Output

Output is stored in a dataset. Each item is information about weather in a location. The items come in two forms - day/night values for daily items and current values for moment items. Example for day/night values:

```
{
  "city": "Třeboň",
  "state": "South Bohemia",
  "country": "Czech Republic",
  "zipCode": "379 01",
  "time": "2020-08-15T07:00:00+0200",
  "temperature": "24/16",
  "forecast": "Thunderstorms/Scattered Thunderstorms",
  "humidity": "79/88",
  "windDirection": "W/WNW",
  "windSpeed": "9/6"
}
```

Example for current values:
```
{
  "city": "Třeboň",
  "state": "South Bohemia",
  "country": "Czech Republic",
  "zipCode": "379 01",
  "time": "2020-08-12T19:00:00+0200",
  "temperature": 27,
  "forecast": "Sunny",
  "humidity": 45,
  "windDirection": "E",
  "windSpeed": 7
}
```

## Extend output function

You can use this function to update the default output of this actor. This function gets a JQuery handle `$` as an argument so you can choose what data from the page you want to scrape. The output from this will function will get merged with the default output.

The return value of this function has to be an object!

You can return fields to achive 3 different things:
- Add a new field - Return object with a field that is not in the default output
- Change a field - Return an existing field with a new value
- Remove a field - Return an existing field with a value `undefined`

## Custom map function

You can use this function to change the output of each of the rows that you will get from this actor. This function gets each of the rows on the output as an argument so you can change the formatting, pick which attributes you want to get as the output. The output from this function will be converted into the format you will return.

The return value of this function has to be an object!

You can return fields to achive 3 different things:
- Add a new field - Return object with a field that is not in the default output
- Change a field - Return an existing field with a new value
- Remove a field - Return an existing field with a value `undefined`


```
(object) => {
    return {
        title: object.title.replace(" - The Weather Channel | Weather.com", "") ,
        windSpeed: 7 + "knots",
    }
}
```
This example will change the output in the following way:

```
{
  "title": "Třeboň, South Bohemia, Czech Republic 10-Day Weather Forecast"
  "windSpeed": "7 knots"
}
```

Note that all the data are scraped from today page (eg. `https://weather.com/weather/today/l/81cbe8a06fd80171651aef7a414bce1e599aa05082d82f4e319f94b4b60602e0`).

## During the Run

During the run, the actor will output messages letting you know what is going on. Each message always contains a short label specifying which page from the provided list is currently specified.
When items are loaded from the page, you should see a message about this event with a loaded item count and total item count for each page.

If you provide incorrect input to the actor, it will immediately stop with a failure state and output an explanation of what is wrong.

## Weather Scraper Export

During the run, the actor stores results into a dataset. Each item is a separate item in the dataset.

You can manage the results in any language (Python, PHP, Node JS/NPM). See the FAQ or <a href="https://www.apify.com/docs/api" target="blank">our API reference</a> to learn more about getting results from this Weather Scraper actor.

## Contact
Please visit us through [epctex.com](https://epctex.com) to see all the products that are available for you. If you are looking for any custom integration or so, please reach out to us through the chat box in [epctex.com](https://epctex.com). In need of support? [business@epctex.com](mailto:business@epctex.com) is at your service.
