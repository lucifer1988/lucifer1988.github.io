---
layout: post
title: "React Native初探1"
date: 2015-10-20 17:29:08 +0800
comments: true
categories: iOS
published: true 
---

React Native是Facebook最近推出的一个框架，让开发者通过JavaScript来完成iOS或Android的Native App，类似的方案好像几年前就出现过，比如PhoneGap，但经过市场检验，其App的用户体验，尤其是UI方面，始终较Native App相距甚远，那么这次Facebook带来的解决方案又有什么不同呢？

<!--more-->

## React Native特点

1. 使用React Native后，你的App的逻辑部分是用JavaScript完成的，而UI则是完全native的，所以也不用担心H5带来的体验下降。
2. React Native还为用户界面构建带来了一种全新的函数式构建方案，App的UI会用与app的状态有关的函数来呈现。
3. React Native的核心思路是将响应式编程引入移动开发，这并不像之前PhoneGap倡导的*write-once,run-anywhere*，而是*learn-once,write-anywhere*。
4. Swift中Apple倡导使用函数式编程来完成算法和业务逻辑，但是构建UI仍然是基于UIKit，并没有实现函数式实现，而React则引入了UI层的函数式编程。

## React Native环境搭建

React Native的源码开源在[GitHub](https://github.com/facebook/react-native)上，不过如果只是开发使用，则推荐使用commend line interface（CLI）来创建项目。  
1.React Native使用Node.js，JavaScript的runtime，来创建JS代码。这里推荐使用Homebrew来安装Node.js。  

```ruby
brew install node
```

2.然后安装watchman，一个Facebook开发的文件监听器，React Native通过它来监视代码的改动并适时编译，类似在Xcode中保存一次文件，便会执行一次build。  

```ruby
brew install watchman
```

3.然后利用安装好的Node.js带的Node Package Manager来安装React Native CLI工具。  

```ruby
npm install -g react-native-cli
```

4.这样环境就搭建完毕了，然后在目标文件夹，利用React Native的CLI创建新的项目。  

```ruby
react-native init YourAppFolderName
```

## Hello Recact Native

1.下面来创建一个“Hello World”的小程序，首先打开**index.ios.js**文件，删除测试代码，先添加以下代码，这段代码是开启了Strict Mode，主要为了提高JS的错误处理和禁用一些JS的语言特性。  

```javascript
'use strict'
```

2.然后继续添加，这其实是导入了react-native模块，并将其赋值给了React，类似于#import或include。  

```javascript
var React = require("react-native")
```

3.再添加一个样式，React Native使用了CSS来定义UI的样式，这和web开发是一致的。  

```javascript
 var styles = React.StyleSheet.create({
 	text:{
 		color:'black',
 		backgroundColor:'white',
 		fontSize:30,
 		margin:80
 	}
 })
```

4.创建一个JS的类，Class是在ES6中引入的，但web开发为了兼容旧版浏览器，一般不会使用类，React Native是基于JavaScriptCore，可以放心使用JS的新特性，而不用担心浏览器兼容的问题。PropertyFinderApp扩展了React.Component，它是React UI的基本构建块，包含了不可变的Properties、可变的状态变量和用于渲染的方法，这里由于程序简单，只有一个渲染方法。  

```javascript
class PropertyFinderApp extends React.Component{
	render(){
		return React.createElement(React.Text, {style:styles.text}, "Hello World!");
	}
}
```

5.React.Component不是UIKit类，可以说是UIKit类的替代品，框架来负责将React components来转换为Native UI。最后再添加程序的入口，这里需要提供root component，也就是我们上面定义的PropertyFinderApp。  

```javascript
React.AppRegistry.registerComponent("RANTest", function(){return PropertyFinderApp});
```

6.然后运行程序，你会发现JS代码已经转化为Native元素，完全没有网页元素出现。  

## React Native运作原理

1.先来看一下OC的程序加载后做了什么，一个类为RCTRootView的对象被创建，它负责加载JS程序和渲染视图，它通过*http://localhost:8081/index.ios.bundle?platform=ios&dev=true*来加载JS。  

```objectivec
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  NSURL *jsCodeLocation;
  jsCodeLocation = [NSURL URLWithString:@"http://localhost:8081/index.ios.bundle?platform=ios&dev=true"];
  RCTRootView *rootView = [[RCTRootView alloc] initWithBundleURL:jsCodeLocation
                                                      moduleName:@"RANTest"
                                               initialProperties:nil
                                                   launchOptions:launchOptions];
  self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];
  UIViewController *rootViewController = [[UIViewController alloc] init];
  rootViewController.view = rootView;
  self.window.rootViewController = rootViewController;
  [self.window makeKeyAndVisible];
  return YES;
}
```

2.当你运行程序时，会打开一个终端窗口，就是为了开启一个packager和server来处理上述请求，在浏览器打开这一URL，也可以看到JS代码。  
3.当app启动时，这些代码被载入，并被JavaScriptCore framework执行，将各个component载入，然后构建Native UIKit视图。  

## Hello World JSX

1.为了增加可读性和提高可维护性，可以使用HTML样式的JSX，也就是JavaScript syntax extension。  

```javascript
return <React.Text style={styles.text}>Hello World(Again)</React.Text>;
```

## A Search React Native App1

### Adding Navigation

1.这一节构建一个OC的NavgationController，将之前的PropertyFinderApp类改为HelloWorld，然后新定义一个PropertyFinderApp类，initialRoute定为HelloWorld，routing技术是web开发中定义导航结构，即哪个页面（或route）对应哪个URL。  

```javascript
class PropertyFinderApp extends React.Component {
  render() {
    return (
      <React.NavigatorIOS
        style={styles.container}
        initialRoute={ {
          title: 'Property Finder',
          component: HelloWorld,
        } }/>
    );
  }
}
```

2.然后添加container样式，这样，一个简单的导航控制器就完成了。  

```javascript
var styles = React.StyleSheet.create({
	text:{
		color:'black',
		backgroundColor:'white',
		fontSize:30,
		margin:80
	},
	container:{
		flex:1
	}
})
```

### Building the Search Page

1.这一节介绍如何添加自定义一个Search页面，并在其他文件中进行引用。首先新建一个SearchPage.js文件，并在文件中添加代码。  

```javascript
'use strict'
var React = require('react-native');
var {
	StyleSheet,
	Text,
	TextInput,
	View,
	TouchableHighlight,
	ActivityIndicatorIOS,
	Image,
	Component
} = React;
```

2.这里使用了destructuring assignment，可以通过一行代码将多个对象属性一次性输出并将他们赋值给多个变量，这样你可以在之后的代码去掉React前缀，例如直接引用StyleSheet而不是React.StyleShet，这一技术在[修改数组](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)时也经常用到，有点类似Swift的元组取值模式。  

```javascript
var foo = ["one", "two", "three"];
// without destructuring
var one   = foo[0];
var two   = foo[1];
var three = foo[2];
// with destructuring
var [one, two, three] = foo;
```

3.然后创建CSS样式，并创建SearchPage component，语法依然使用了JSX的结构。  

```javascript
var styles = StyleSheet.creat({
	description:{
		marginBottom:20,
		fontSize:18.
		textAlign:'center'.
		color:'#656565'
	},
	container:{
		padding:30,
		marginTop:65,
		alignItems:'center'
	}
});
```

```javascript
class SearchPage extends Component {
	render() {
		return(
		 <View style={styles.container}>
		  <Text style={styles.description}>
		   Search for houses to buy!
		  </Text>
		  <Text style={styles.description}>
		   Search by place-name, postcode or search near your location.
		  </Text>
		 </View>
		);
	}
}
```

4.最后在文件结尾将SearchPage作为输出，并在index.ios.js中导入SearchPage，然后将之前render方法中的PropertyFinderApp类中的initialRoute更新。  

```javascript
module.exports = SearchPage;
```

```javascript
var SearchPage = require('./SearchPage');
```

```javascript
initialRoute={ {
  title: 'Property Finder',
  component: SearchPage,
} }
```

### Styling with Flexbox

1.flexbox是CSS最近加入的新特性，用于页面的布局（类似Autolayout），React Native使用了[css-layout](http://https://github.com/facebook/css-layout)库，该库是Facebook自己开发的一个使用了flexbox标准的JS库，而这一标准对于C(iOS)和Java(Android)都是可接受的，这里有一篇专门讲解[flexbox layout to SVG](http://blog.scottlogic.com/2015/02/02/svg-layout-flexbox.html)的文章，也是该作者写的。  
2.在这个app中，container默认是列方向布局，也就是垂直方向布局（这与Android的布局方式是相同的），同时container也可以决定他的子视图的布局方向。  

```javascript
<View style={styles.flowRight}>
  <TextInput
    style={styles.searchInput}
    placeholder='Search via name or postcode'/>
  <TouchableHighlight style={styles.button}
      underlayColor='#99d9f4'>
    <Text style={styles.buttonText}>Go</Text>
  </TouchableHighlight>
</View>
<TouchableHighlight style={styles.button}
    underlayColor='#99d9f4'>
  <Text style={styles.buttonText}>Location</Text>
</TouchableHighlight>
```

3.添加新的样式，记得在每个样式后要添加逗号分开，flex值是为了划分同一container下子视图的占位比，如这里的Go button和input view的flex分别为1和4，那么就按照1:4来划分，另外，这里的button是使用了TouchableHighlight。  

```javascript
flowRight: {
  flexDirection: 'row',
  alignItems: 'center',
  alignSelf: 'stretch'
},
buttonText: {
  fontSize: 18,
  color: 'white',
  alignSelf: 'center'
},
button: {
  height: 36,
  flex: 1,
  flexDirection: 'row',
  backgroundColor: '#48BBEC',
  borderColor: '#48BBEC',
  borderWidth: 1,
  borderRadius: 8,
  marginBottom: 10,
  alignSelf: 'stretch',
  justifyContent: 'center'
},
searchInput: {
  height: 36,
  padding: 4,
  marginRight: 5,
  flex: 4,
  fontSize: 18,
  borderWidth: 1,
  borderColor: '#48BBEC',
  borderRadius: 8,
  color: '#48BBEC'
}
```

4.然后添加一张图片，图片资源依然要添加到Xcode的Images.xcassets，使用require('image!house')来加载图片，  

```javascript
<Image source={require('image!house')} style={styles.image}/>
```

```javascript
image: {
  width: 217,
  height: 138
}
```

## A Search React Native App2

### Adding Component State

1.这一节，让我们来处理TextInput的输入，首先，我们来初始化SearchPage Component，下列代码添加到render()之前，这里有了新变量state以及searchString，并对TextInput赋该值。    

```javascript
constructor(props) {
  super(props);
  this.state = {
    searchString: 'london'
  };
}
```

```javascript
<TextInput
  style={styles.searchInput}
  value={this.state.searchString}
  placeholder='Search via name or postcode'/>
```

2.然后在SearchPage Class添加一个方法，作为TextInput的回调，并将其与TextInput绑定，这一过程在OC中是以delegate的形式实现的，需要说明下的是这里的this是指向所在component的实例的指针。  

```javascript
onSearchTextChanged(event) {
  console.log('onSearchTextChanged');
  this.setState({ searchString: event.nativeEvent.text });
  console.log(this.state.searchString);
}
```

```javascript
<TextInput
  style={styles.searchInput}
  value={this.state.searchString}
  onChange={this.onSearchTextChanged.bind(this)}
  placeholder='Search via name or postcode'/>
```

3.通过实验TextInput我们发现，每次TextIput的状态发生改变，整个component就会重新render一次，这一机制将渲染逻辑和与UI有关的状态改变彻底分开。在大部分UI框架中，一般都是开发者根据app状态改变来手动刷新UI（比如OC或Swift），或者使用隐式链接来绑定app的状态和UI刷新完成自动刷新（比如ReactiveCocoa），但是在React Native中，你不用再去手动处理这些逻辑，整个UI就是app状态的一个函数表示！这就是响应式编程的核心理念。  
4.不过你可能会担心效率问题，频繁刷新整个UI当然是不明智的，React在每次刷新时，它会从render方法获取整个视图树，然后与现在的UIKit视图进行比较，比较的结果就是一个简单的更新表，React按照这个表去更新当前视图，所以只有需要更新的UI才会去更新。  
5.这一理念的应用，将虚拟DOM和一致性引入了App开发，也是React的独特之处。  

### Initiating a Search

1.这一节为Search页面添加搜索功能，首先在state中加入isLoading变量，再在render中添加spinner变量告知用户搜索在进行，它依据isLoading变量来添加一个spinner或空视图，并将{spinner}加入return方法。  

```javascript
this.state = {
  searchString: 'london',
  isLoading: false
};
```

```javascript
var spinner = this.state.isLoading ?
  ( <ActivityIndicatorIOS
      hidden='true'
      size='large'/> ) :
  ( <View/>);
...
{spinner}
```

2.在Go Button绑定onPress事件回调，并添加回调方法，注意Javascript的类没有访问器，所以也没有私有方法，所以一般用*_*前缀来标识私有方法。  

```javascript
onPress={this.onSearchPressed.bind(this)}
```

```javascript
_executeQuery(query) {
  console.log(query);
  this.setState({ isLoading: true });
}
onSearchPressed() {
  var query = urlForQueryAndPage('place_name', this.state.searchString, 1);
  this._executeQuery(query);
}
```

3.在SearchPage外单独定义urlForQueryAndPage()方法，这里做了URL的拼接，用到了[JS的方法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions) 。   

```javascript
function urlForQueryAndPage(key, value, pageNumber) {
  var data = {
      country: 'uk',
      pretty: '1',
      encoding: 'json',
      listing_type: 'buy',
      action: 'search_listings',
      page: pageNumber
  };
  data[key] = value;
  
  var querystring = Object.keys(data)
    .map(key => key + '=' + encodeURIComponent(data[key]))
    .join('&');
 
  return 'http://api.nestoria.co.uk/api?' + querystring;
};
```

4.=>是JS中对函数指针的缩写，可理解为OC中的block，这里先用map将其原字典的keys映射为新的数组，然后用&相连，产生URL的参数String。    

```javascript
var a = [
  "Hydrogen",
  "Helium",
  "Lithium",
  "Beryl­lium"
];
var a2 = a.map(function(s){ return s.length });
var a3 = a.map( s => s.length );
```

### Performing an API Request

1.在state添加message变量，并在render添加Text，用以显示请求错误信息，并在_excuteQuery()中添加请求的代码。  

```javascript
this.state = {
  searchString: 'london',
  isLoading: false,
  message: ''
};
```

```javascript
<Text style={styles.description}>{this.state.message}</Text>
```

2.请求代码使用了fetch函数，这是[Fetch API](https://fetch.spec.whatwg.org)中的函数，相比XMLHttpRequest，有很大提升，使用了promise规范。  

```javascript
fetch(query)
  .then(response => response.json())
  .then(json => this._handleResponse(json.response))
  .catch(error => 
     this.setState({
      isLoading: false,
      message: 'Something bad happened ' + error
   }));
```

3.success的回调处理，先对response code做了判断，然后打印出了listings的长度，可以看出JSON在JS开发中是直接使用的，而省去了转化为Model的步骤。    

```javascript
_handleResponse(response) {
  this.setState({ isLoading: false , message: '' });
  if (response.application_response_code.substr(0, 1) === '1') {
    console.log('Properties found: ' + response.listings.length);
  } else {
    this.setState({ message: 'Location not recognized; please try again.'});
  }
}
```

### Displaying the Results

1.新建SearchResult.js，新建SearchResults component，代码中使用到了ListView，类似OC的UITableView，通过dataSource来提供数据源，renderRow来渲染每个cell。   

```javascript
class SearchResults extends Component {
 
  constructor(props) {
    super(props);
    var dataSource = new ListView.DataSource(
      {rowHasChanged: (r1, r2) => r1.guid !== r2.guid});
    this.state = {
      dataSource: dataSource.cloneWithRows(this.props.listings)
    };
  }
 
  renderRow(rowData, sectionID, rowID) {
    return (
      <TouchableHighlight
          underlayColor='#dddddd'>
        <View>
          <Text>{rowData.title}</Text>
        </View>
      </TouchableHighlight>
    );
  }
 
  render() {
    return (
      <ListView
        dataSource={this.state.dataSource}
        renderRow={this.renderRow.bind(this)}/>
    );
  }
}
```

2.构建数据源时，提供了一个方法来比较row之间的id是否相同，ListView在更新时调用它，来确定数据源是否改变，本例中通过数据的guid来达到这个目的，然后在SearchPage的_handleResponse中添加导航方法。    

```javascript
this.props.navigator.push({
  title: 'Results',
  component: SearchResults,
  passProps: {listings: response.listings}
});
```

### A Touch of Style

1.添加样式，更新renderRow()方法，price为了去掉GBP后缀，做了字符串裁剪，同时这次的Image数据源为url，JS可直接赋值，而React Native会自动后台一步下载。  

```javascript
var styles = StyleSheet.create({
  thumb: {
    width: 80,
    height: 80,
    marginRight: 10
  },
  textContainer: {
    flex: 1
  },
  separator: {
    height: 1,
    backgroundColor: '#dddddd'
  },
  price: {
    fontSize: 25,
    fontWeight: 'bold',
    color: '#48BBEC'
  },
  title: {
    fontSize: 20,
    color: '#656565'
  },
  rowContainer: {
    flexDirection: 'row',
    padding: 10
  }
});
```

```javascript
renderRow(rowData, sectionID, rowID) {
  var price = rowData.price_formatted.split(' ')[0];
  
  return (
    <TouchableHighlight onPress={() => this.rowPressed(rowData.guid)}
        underlayColor='#dddddd'>
      <View>
        <View style={styles.rowContainer}>
          <Image style={styles.thumb} source={ { uri: rowData.img_url } } />
          <View  style={styles.textContainer}>
            <Text style={styles.price}>£{price}</Text>
            <Text style={styles.title} 
                  numberOfLines={1}>{rowData.title}</Text>
          </View>
        </View>
        <View style={styles.separator}/>
      </View>
    </TouchableHighlight>
  );
}
```

2.然后添加点击cell的回调，可以看到数据源始终还是最原始的数据源，做了一步过滤操作（因为这里的cell没有index的概念，所以只能过滤，但是必须保证guid唯一才可以）。    

```javascript
rowPressed(propertyGuid) {
  var property = this.props.listings.filter(prop => prop.guid === propertyGuid)[0];
}
```

### Property Details View

1.添加详情页，PropertyView，首先对房屋的配置信息做了整理，然后就是常规的视图布局。  

```javascript
class PropertyView extends Component {
  render() {
    var property = this.props.property;
    var stats = property.bedroom_number + ' bed ' + property.property_type;
    if (property.bathroom_number) {
      stats += ', ' + property.bathroom_number + ' ' + (property.bathroom_number > 1 ? 'bathrooms' : 'bathroom');
    }
    
    var price = property.price_formatted.split(' ')[0];
    
    return (
      <View style={styles.container}>
        <Image style={styles.image} 
            source={ {uri: property.img_url} } />
        <View style={styles.heading}>
          <Text style={styles.price}>£{price}</Text>
          <Text style={styles.title}>{property.title}</Text>
          <View style={styles.separator}/>
        </View>
        <Text style={styles.description}>{stats}</Text>
        <Text style={styles.description}>{property.summary}</Text>
      </View>
    );
  }
}
```

2.最后从SearchResults，推入PropertyView。  

```javascript
rowPressed(propertyGuid) {
  var property = this.props.listings.filter(prop => prop.guid === propertyGuid)[0];
 
  this.props.navigator.push({
    title: "Property",
    component: PropertyView,
    passProps: {property: property}
  });
}
```

### Geolocation Search

1.增加自动定位并搜索周边房屋的功能，在SearchPage的Location按钮添加该功能，前提在Xcode的工程的Plist中添加NSLocationWhenInUseUsageDescription来添加请求允许定位的描述。  

```javascript
onPress={this.onLocationPressed.bind(this)}
```

```javascript
onLocationPressed() {
  navigator.geolocation.getCurrentPosition(
    location => {
      var search = location.coords.latitude + ',' + location.coords.longitude;
      this.setState({ searchString: search });
      var query = urlForQueryAndPage('centre_point', search, 1);
      this._executeQuery(query);
    },
    error => {
      this.setState({
        message: 'There was a problem with obtaining your location: ' + error
      });
    });
}
```


