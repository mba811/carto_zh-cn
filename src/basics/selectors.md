### 3.2 样式选择器

在CartoCSS中，地图样式通过一系列描述样式的规则来表达。这些样式规则被模块化的组织成**样式块**，作用于地图上的各种要素对象。每个样式块都由一对大括号`{}`包围，其中包含了若干条用于描述样式的属性和值。**样式选择器**的作用就是指明某个样式块是作用于哪个图层，或者进一步限定这些样式在特定图层中的作用范围是哪些要素对象。因此从本质上说，样式选择器其实就是描述了样式块的作用域。

样式选择器可以有三种不同的形式：图层标识、图层类别，以及过滤器。其中过滤器还可以分为缩放级别、数值、文本和正则表达式等三种具体类型。

#### 图层标识

通过图层的唯一标识（ID）来将样式块的作用范围限定在某一个或几个图层上。对于多个图层共用同一样式块的情况，应该将多个图层标识以逗号隔开。

	
	#layer_name {
	  // 样式描述
	}
	#layer_1,
	#layer_2 {
	  // 这里的样式将被应用于layer_1和layer_2两个图层
	}
	

#### 图层类别

当需要对多个图层定义样式时，除了可以采用之前提到的将图层标识全部列出的方法以外，还可以使用图层类别来实现，也就是在这些图层标识的后面都加上一个类别名后缀，还以上面的两个图层为例，可以为它们增加一个同一个类别得到`layer_1.roads`和`layer_2.roads`两个新的标识，然后通过对`.roads`进行样式定义来实现与之前同样的效果。

	
	.roads {
	  // 这里定义的样式会被应用到所有
	  // 类别后缀为'roads'的图层上
	}
	

#### 过滤器

除了用图层的标识或类别来限定样式块的作用范围以外，还可以进一步用基于条件表达式的过滤器在图层内部筛选出需要应用样式的那些要素对象。这些过滤器使样式块与图层内要素对象的各种文本或数值类型的属性数据建立起关联，从而实现了_条件样式_。过滤器在制图过程中非常实用，举个例子：一个内容为道路网的线要素图层中通常是将各种不同等级的道路都包含在内，而利用过滤器，我们就可以为不同等级的道路配置不同的样式。

过滤器需要被置于一对中括号`[]`中，跟在图层选择器（标识或类别）的后面，或者是嵌套的写在一个更大的样式块中。

##### 缩放级别过滤器

将样式的作用范围限制在特定的地图缩放级别上。在下面的例子中，样式块中定义的样式只在地图缩放到0级时发挥作用。0级是观察点距离地球表面目标区域最远的级别，这个级别看到的通常就是世界地图视图。随着缩放级别变大，观察点也逐渐向地球表面“推进”，位于当前视图中的部分地图比例尺不断变大，分辨率逐渐升高，就如同将相机镜头推向长焦端，因此这个过程的英文是“zoom in”，而相反的过程为“zoom out”。

	
	#layer[zoom=0] { /* style */ }
	

还可以定义缩放级别的范围：

	
	#layer[zoom>=4][zoom<=10] { /* style */ }
	

缩放级别过滤器支持6种关系运算符：`=`（等于，**注意不是双等号**）、`>`（大于）、`<`（小于）、`>=`（大于等于）、`<=`（小于等于）和`!=`（不等于）。

过滤器可以以嵌套的形式出现在样式块内部。例如，在下面这段CartoCSS代码中，线要素会在4级到10级之间被绘制成2个像素宽的红色线条，而在8级、9级和10级，线宽会被重新修正为3、4和5个像素。

	
	#layer[zoom>=4][zoom<=10] {
	  line-color: red;
	  line-width: 2;
	  [zoom=8] { line-width: 3; }
	  [zoom=9] { line-width: 4; }
	  [zoom=10] { line-width: 5; }
	}
	

##### 数值型过滤器

关系运算符还可以用于对图层中的数值型属性字段进行过滤。举个例子，在一个表示城市信息的点要素图层中，有一个用于记录每个城市人口数据的字段，那么我们就可以在该字段上应用数值型过滤器，实现“只有人口在一百万以上的城市才被标注在地图上”的效果：

	
	#cities[population>1000000] {
	  text-name: [name];
	  text-face-name: 'Open Sans Regular';
	}
	

这种数值型过滤器还可以和之前介绍的缩放级别过滤器结合使用。还以城市和人口的数据为例，将两种过滤器结合使用可以实现在不同的缩放级别上显示不同人口规模的城市。

	
	#cities {
	  [zoom>=4][population>1000000],
	  [zoom>=5][population>500000],
	  [zoom>=6][population>100000] {
	    text-name: [name];
	    text-face-name: 'Open Sans Regular';
	  }
	}
	

与缩放级别过滤器相同，数值型过滤器中也同样支持范围过滤：

	
	#cities[population>100000][population<2000000] { /* styles */ }
	

##### 文本型过滤器

对于文本类型的属性字段，同样也可以应用过滤器。通过使用等号运算符`=`，可以实现精确匹配，或者通过不等号运算符`!=`来得到相反的结果。与前两种过滤器不同的是，文本型过滤器关系表达式中的文本值必须要有双引号或单引号。

As an example, look at the roads layer in Mapbox Streets (the default vector tile source in Mapbox Studio). It contains a field called class, and each value for this field is one of just a few options such as “motorway”, “main”, and “street”. This makes it a good column to filter on for styling.

举个例子，在一个表示道路网的线要素图层中包含了一个名为`class`的文本型字段，该字段取值为`"motoway"`，`"main"`或者`"street"`，用于表示每条道路的类别。那么在制图过程中，用户就可以在该字段上应用文本型过滤器，从而实现对不同类型的道路使用不同的样式进行渲染：

	
	#roads {
	  [class='motorway'] {
	    line-width: 4;
	  }
	  [class='main'] {
	    line-width: 2;
	  }
	  [class='street'] {
	    line-width: 1;
	  }
	}
	

如果要对所有不是`motoway`的道路进行样式配置，那么还可以使用不等号：

	
	#roads[class!='motorway'] { /* style */ }
	

##### 正则表达式过滤器

_注意：正则表达式过滤器作为一种高级过滤功能，可能会对制图渲染性能带来负面影响。_

用户还可以通过基于模式匹配的正则表达式进行过滤，正则表达式过滤器的运算符是`=~`。在下面的例子中，正则表达式过滤器会匹配所有`class`字段中以`motoway`开头的要素记录，像`motoway`、`motoway_link`都会被匹配上。

	
	#roads[class=~'motorway.*'] { /* style */ }
	

在上面的例子中，`.`表示任意字符，`*`表示出现任意多次，因而`.*`合在一起就表示由任意字符组成的任意长度的字符串。

#### 参考文献

1. Mapbox, [Mapbox Studio Styling Selectors](https://www.mapbox.com/mapbox-studio/styling-selectors/)