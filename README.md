# SwiftCharts

[![Version](https://img.shields.io/cocoapods/v/SwiftCharts.svg?style=flat)](http://cocoadocs.org/docsets/Charts)
[![License](https://img.shields.io/cocoapods/l/SwiftCharts.svg?style=flat)](http://cocoadocs.org/docsets/Charts)

Easy to use and highly customizable charts library for iOS

##### Features:
- Bars - plain, stacked, grouped, horizontal, vertical
- Scatter
- Line / Multiple lines
- Areas
- Bubble
- Multiple axis
- Custom units
- Custom views
- Multiple labels (x axis)
- Candlestick
- Cubic line
- Complex interactivity support
- Easy extensibility
- **Easy to use**: no learning curve, no complicated settings - just assemble chart using well defined components.

Swift 1.2, iOS 7/8. Swift 2.0 branch. So far everything seems to be working in iOS9 (beta 2), except stacked bars. Fix coming soon.

[Video](https://www.youtube.com/watch?v=bD6uDF-KckM) (this will be updated only on new tags - check the [changelog](CHANGELOG.md) for possible new examples in master)

##### Screenshots:

![ScreenShot](https://raw.github.com/i-schuetz/SwiftCharts/master/Screenshots/IMG_0102.jpeg)
![ScreenShot](https://raw.github.com/i-schuetz/SwiftCharts/master/Screenshots/IMG_0022.jpeg)
![ScreenShot](https://raw.github.com/i-schuetz/SwiftCharts/master/Screenshots/IMG_0023.jpeg)
![ScreenShot](https://raw.github.com/i-schuetz/SwiftCharts/master/Screenshots/IMG_0024.jpeg)
![ScreenShot](https://raw.github.com/i-schuetz/SwiftCharts/master/Screenshots/IMG_0025.jpeg)
![ScreenShot](https://raw.github.com/i-schuetz/SwiftCharts/master/Screenshots/IMG_0026.jpeg)
![ScreenShot](https://raw.github.com/i-schuetz/SwiftCharts/master/Screenshots/IMG_0101.jpeg)
![ScreenShot](https://raw.github.com/i-schuetz/SwiftCharts/master/Screenshots/IMG_0027.jpeg)
![ScreenShot](https://raw.github.com/i-schuetz/SwiftCharts/master/Screenshots/IMG_0028.jpeg)
![ScreenShot](https://raw.github.com/i-schuetz/SwiftCharts/master/Screenshots/IMG_0029.jpeg)
![ScreenShot](https://raw.github.com/i-schuetz/SwiftCharts/master/Screenshots/IMG_0031.jpeg)
![ScreenShot](https://raw.github.com/i-schuetz/SwiftCharts/master/Screenshots/IMG_0033.jpeg)
![ScreenShot](https://raw.github.com/i-schuetz/SwiftCharts/master/Screenshots/IMG_0034.jpeg)
![ScreenShot](https://raw.github.com/i-schuetz/SwiftCharts/master/Screenshots/IMG_0037.jpeg)
![ScreenShot](https://raw.github.com/i-schuetz/SwiftCharts/master/Screenshots/IMG_0038.jpeg)
![ScreenShot](https://raw.github.com/i-schuetz/SwiftCharts/master/Screenshots/IMG_0039.jpeg)
![ScreenShot](https://raw.github.com/i-schuetz/SwiftCharts/master/Screenshots/IMG_0040.jpeg)
![ScreenShot](https://raw.github.com/i-schuetz/SwiftCharts/master/Screenshots/IMG_0041.jpeg)


##### Installation

Add to your podfile:
```ruby
use_frameworks!
pod 'SwiftCharts', '~> 0.2'
```
Or to use the Swift 2.0 branch:
```ruby
use_frameworks!
pod 'AFNetworking', :git => 'https://github.com/i-schuetz/SwiftCharts.git', :branch => 'swift2.0'
```

And then:
```ruby
pod install
```

Import the framework in your code:
```swift
import SwiftCharts
```

####Concept:

- Layer architecture, which makes it extremely easy to customize charts, create new types, combine existing ones and add interactive elements.

- Creation of views via a generator function, which makes it easy to use custom views in any layer.

##### Main Components:

##### 1. Layers:

A chart is the result of composing layers together. Everything is a layer - axis, guidelines, dividers, line, circles, etc. The idea is to have losely coupled components that can be easily changed and combined. This is for example the structure of a basic chart, which shows a line with circles:

![ScreenShot](https://raw.github.com/i-schuetz/SwiftCharts/master/Screenshots/layers.png)


Code:

```swift
let labelSettings = ChartLabelSettings(font: ExamplesDefaults.labelFont)

let chartPoints = [(2, 2), (3, 1), (5, 9), (6, 7), (8, 10), (9, 9), (10, 15), (13, 8), (15, 20), (16, 17)].map{ChartPoint(x: ChartAxisValueInt($0.0), y: ChartAxisValueInt($0.1))}

let xValues = Array(stride(from: 2, through: 16, by: 2)).map {ChartAxisValueInt($0, labelSettings: labelSettings)}
let yValues = Array(stride(from: 0, through: 20, by: 2)).map {ChartAxisValueInt($0, labelSettings: labelSettings)}

let xModel = ChartAxisModel(axisValues: xValues, axisTitleLabel: ChartAxisLabel(text: "Axis title", settings: labelSettings))
let yModel = ChartAxisModel(axisValues: yValues, axisTitleLabel: ChartAxisLabel(text: "Axis title", settings: labelSettings))
let chartFrame = ExamplesDefaults.chartFrame(self.view.bounds)

let chartSettings = ExamplesDefaults.chartSettings
let coordsSpace = ChartCoordsSpaceLeftBottomSingleAxis(chartSettings: chartSettings, chartFrame: chartFrame, xModel: xModel, yModel: yModel)
let (xAxis, yAxis, innerFrame) = (coordsSpace.xAxis, coordsSpace.yAxis, coordsSpace.chartInnerFrame)

// create layer with line
let lineModel = ChartLineModel(chartPoints: chartPoints, lineColor: UIColor(red: 0.4, green: 0.4, blue: 1, alpha: 0.2), lineWidth: 3, animDuration: 0.7, animDelay: 0)
let chartPointsLineLayer = ChartPointsLineLayer(xAxis: xAxis, yAxis: yAxis, innerFrame: innerFrame, lineModels: [lineModel])

// view generator - creates circle view for each chartpoint
let circleViewGenerator = {(chartPointModel: ChartPointLayerModel, layer: ChartPointsLayer, chart: Chart) -> UIView? in
    return ChartPointCircleView(center: chartPointModel.screenLoc, size: CGSizeMake(20, 20), settings: ChartPointCircleViewSettings(animDuration: 0.5))
}
// create layer that uses the view generator
let chartPointsCircleLayer = ChartPointsViewsLayer(xAxis: xAxis, yAxis: yAxis, innerFrame: innerFrame, chartPoints: chartPoints, viewGenerator: circleViewGenerator, displayDelay: 0, delayBetweenItems: 0.05)

// create layer with guidelines
var settings = ChartGuideLinesDottedLayerSettings(linesColor: UIColor.blackColor(), linesWidth: ExamplesDefaults.guidelinesWidth, axis: .XAndY)
let guidelinesLayer = ChartGuideLinesDottedLayer(xAxis: xAxis, yAxis: yAxis, innerFrame: innerFrame, settings: settings)

let chart = Chart(
    frame: chartFrame,
    layers: [
        xAxis,
        yAxis,
        guidelinesLayer,
        chartPointsLineLayer,
        chartPointsCircleLayer
    ]
)

self.view.addSubview(chart.view)
self.chart = chart
```

Layers are semantic units that can add views to the chart, or can simply draw in the chart's context for a better performance. Which makes more sense depends on the requirements.

##### 2. View generators:

View based layers will use a generator function to generate chart point views. This function receives the complete state of each chartpoint (model data, screen location) and produces an UIView, allowing any type of customization.

##### Hello world:

There's a [hello world](Examples/Examples/Examples/HelloWorld.swift) included in the examples, similar to the above code, with a bit more explanations. Change some properties of the generated views, copy paste the chartPointsLineLayer used in the snippet above, and pass it to the chart's layers, to display a line behind the views, and you have already mastered the main concepts!

##### Functional concepts:

This library is rather object oriented, but it makes extensive use of functional concepts like high order functions and (preference for) immutability leading to safer code and better maintainability.


####Note on performance:

If you have a lot of axis labels in your chart it may be necessary to do the calculation of the coordinate space in the background. There can be a small delay which is noticeable during transitions or scrolling. See ScrollExample or MultipleAxesExample example for this.

####Coming in the next releases:

0.3 Performance improvements

0.4 Create well known chart types with little code

###### More documentation also coming soon!


##### Version:

0.2.2

##### Contributing

1. Fork
2. Commit changes to a branch in your fork
3. Push your code and make a pull request

##### Created By:

Ivan Schütz

##### License

SwiftCharts is Copyright (c) 2015 Ivan Schütz and released as open source under the attached [Apache 2.0 license](LICENSE).

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS BE LIABLE FOR ANY CLAIM, DAMAGES OR
OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE,
ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR
OTHER DEALINGS IN THE SOFTWARE.

This is a port to Swift and (massively improved) continuation of an obj-c project which I did while working at eGym GmbH https://github.com/egymgmbh/ios-charts
