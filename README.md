// Define the area of interest: Sambalpur District
var sambalpur = ee.Geometry.Polygon(
[[[83.833, 21.304],
[83.833, 21.261],
[84.005, 21.261],
[84.005, 21.304]]]);

// Load precipitation data (e.g., CHIRPS for 2021 and 2022)
var precipitation = ee.ImageCollection("UCSB-CHG/CHIRPS/DAILY")
.filterBounds(sambalpur)
.filterDate('2021-01-01', '2022-12-31')
.select('precipitation');

// Load evapotranspiration data (e.g., MODIS for 2021 and 2022)
var et = ee.ImageCollection("MODIS/006/MOD16A2")
.filterBounds(sambalpur)
.filterDate('2021-01-01', '2022-12-31')
.select('ET');

// Function to calculate SPI
var computeSPI = function(imageCollection, scale) {
// Reduce the image collection to monthly sums
var monthlyPrecipitation = imageCollection
.map(function(image) {
return image.set('month', image.date().get('month'))
.set('year', image.date().get('year'));
})
.filterDate('2021-01-01', '2022-12-31')
.map(function(image) {
return image.reduceRegions({
collection: ee.FeatureCollection([ee.Feature(sambalpur)]),
reducer: ee.Reducer.sum(),
scale: scale
}).first().set('system:time_start', image.date().millis());
});

// SPI computation placeholder
// In practice, you would calculate SPI based on the precipitation time series
var spi = ee.ImageCollection.fromImages(monthlyPrecipitation);

return spi;
};

// Function to calculate SPEI
var computeSPEI = function(precipitation, et, scale) {
var combined = precipitation.map(function(image) {
var etImage = ee.Image(et.filterDate(image.date().format('2021-01-01')).first());
return image.subtract(etImage).set('system:time_start', image.date().millis());
});

// Reduce the combined data to monthly sums
var monthlySPEI = combined
.map(function(image) {
return image.reduceRegions({
collection: ee.FeatureCollection([ee.Feature(sambalpur)]),
reducer: ee.Reducer.sum(),
scale: scale
}).first().set('system:time_start', image.date().millis());
});

// SPEI computation placeholder
// In practice, you would calculate SPEI based on the combined time series
var spei = ee.ImageCollection.fromImages(monthlySPEI);

return spei;
};

// Compute SPI and SPEI
var spi = computeSPI(precipitation, 5000);
var spei = computeSPEI(precipitation, et, 5000);

// Visualization (example using placeholders)
Map.centerObject(sambalpur, 8);
Map.addLayer(spi.mean(), {min: -3, max: 3, palette: ['blue', 'white', 'red']}, 'SPI');
Map.addLayer(spei.mean(), {min: -3, max: 3, palette: ['blue', 'white', 'red']}, 'SPEI');

// Export results
Export.image.toDrive({
image: spi.mean(),
description: 'SPI_Sambalpur_2021_2022',
scale: 5000,
region: sambalpur
});

Export.image.toDrive({
image: spei.mean(),
description: 'SPEI_Sambalpur_2021_2022',
scale: 5000,
region: sambalpur
})
