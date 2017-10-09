---
title: java注释规范
date: 2017-04-25 20:29:35
tags:
---

# 文档组成部分
一个文档注释由两部分组成：


```
/**
* 描述部分(description) 
*
* 标记部分(block tags)
*/
```
描述部分指具体描述，所谓的标记符号指的是@param, @return, @see之类的。


* 样例

```java
/**
 * Returns an Image object that can then be painted on the screen. 
 * The url argument must specify an absolute {@link URL}. The name
 * argument is a specifier that is relative to the url argument. 
 * <p>
 * This method always returns immediately, whether or not the 
 * image exists. When this applet attempts to draw the image on
 * the screen, the data will be loaded. The graphics primitives 
 * that draw the image will incrementally paint on the screen. 
 *
 * @param url an absolute URL giving the base location of the image
 * @param name the location of the image, relative to the url argument
 * @return the image at the specified URL
 * @see Image
 */
public Image getImage(URL url, String name) {
    try {
        return getImage(new URL(url, name));
    } catch (MalformedURLException e) {
        return null;
    }
}
```

* 需要注意的几点：

    * 第一行以特殊的文档定界符 /** 开头

    * 在描述段落和标记段落之间空一行，描述段落和标记段落必须分开，不能揉在一起，描述段落必须在标记段落之前


# 描述部分(Description)

描述部分的第一行应该是一句对类、接口、方法等的简单描述，这句话最后会被javadoc工具提取并放在索引目录中。

怎么界定第一句话到哪结束了呢？答案是跟在第一个句号（英文标点）之后的tab、空行或行终结符规定了第一句的结尾。

例如下面这句注释，第一句的结尾是Prof.：

```
/**
* This is a simulation of Prof. Knuth's MIX computer.
*/
```
*   除了普通的文本之外，描述部分可以使用：
    * HTML语法标签，例如 <b>xxx</b> 

    * javadoc规定的特殊标签，例如 {@link xxx} 。标签的语法规则是：{@标签名 标签内容}

* 需要注意的地方：

    * 标签在有javadoc工具生成文档时会转化成特殊的内容，比如 {@link URL} 标签会被转化成指向URL类的超链接

    * 如果注释包含多段内容，段与段之间需要用 <p> 分隔，空行是没用的
    * 最后结尾行 */ 和起始行不同，这里只有一个星号
    * 为了避免一行过长影响阅读效果，务必将每行的长度限制在80个字符以内
    * 善用javadoc工具的复制机制避免不必要的注释：
> 如果一个方法覆盖了父类的方法或实现了接口种的方法，那么javadoc工具会在该注释里添加指向原始方法的链接，此外如果新方法没有注释，那么javadoc会把原始方法的注释复制一份作为其注释，但是如果新方法有注释了，就不会复制了。

* 注释风格
    1. 使用 <code>关键字</code> 来强调关键字，建议强调的内容有：java关键字、包名、类名、方法名、接口名、字段名、参数名等
    2. 控制 {@link xxx} 的数量，太多的链接会使文档的可读性很差，因为读者总是跳来跳去。不要出现相同的链接，同样的链接只保留第一个；不要为java自带的内容或是常识性的内容提供链接
    3. 描述一个方法时，应当只保留方法名字，不要附带方法的参数。比如有个方法是add(Object obj)，那么用add指代该方法即可，而不是add(Object obj)
    4. 英文注释可以是短语也可以是句子。如果是句子，首字母要大写，如果是短语，首字母小写。
    5. 英文注释使用第三人称，而不是第二人称。
    
    ```
    /**
    * Gets the label(建议) 
    */

    /**
    * Get the label(不建议)
    */
    
    ```
    6. 方法的注释应该以动词或动词词组开头，因为方法是一个动作。比如：
    
    ```
    /**
    * Gets the label of this button(建议)
    */

    /**
    * This method gets the label(不建议)
    */
    ```
    7. 当描述类、接口、方法这类的概念时，可以不用指名"类"、"接口"、"方法"这些词语，比如：

    ```
    /**
    * A button label (建议)
    */

    /**
    * This field is a button label (不建议)
    */ 
    ```
    8.英文使用this而不是the指代当前类，比如：
    
    ```
    /**
    * Gets the toolkit for this component (建议)
    */

    /**
    * Gets the toolkit for the component (不建议)
    */
    ```
    9. API名应该是能够简单自我说明的，如果文档注释只是简单重复API的名称还不如没有文档，所以文档注释应该至少提供一些额外信息，否则干脆不要注释
    10. 英文注释避免拉丁风格的缩写。比如使用"also knwon as"而不是"aka"， 使用"that is"或"to be specific"而不是"i.e."，使用"for example"而不是"e.g."，使用"in other words"或"namely"而不是"viz."


# 标记部分(tag)

标记部分跟在描述部分之后，且前面必须有一个空行间隔

* 常见标记
    1.  @author  作者，没有特殊格式要求，名字或组织名称都可以
    2.  @version  软件版本号（注意不是java版本号），没有特殊格式要求
    3.  @param  方法参数，格式为： @param 参数名称 参数描述 

        * 可以在参数描述中说明参数的类型
        * 可以在参数名称和参数描述之间添加额外的空格来对齐
        * 破折号或其他标点符号不能出现在参数描述之外的地方
    4.  @return  方法返回值，格式为： @return 返回值描述 ，如果方法没有返回值就不要写@return
    5.  @deprecated 应该告诉用户这个API被哪个新方法替代了，随后用 @see 标记或 {@link} 标记指向新API，比如：
    
    ```java
    /**
    * @deprecated As of JDK 1.1, replaced by
    * {@link #setBounds(int,int,int,int)}
    */
    ```
    6.  @exception 包含方法显式抛出的检查异常(Checked Exception)，至于非显示抛出的其他异常(Unchecked Exception)，除非特别有必要，否则就别写了。一个原则就是，只记录可控的问题，对于不可控的或不可预测的问题，不要往上面写。
    
* 注释风格
    1. 按照如下顺序提供标记
    ```
    @author（只出现在类和接口的文档中）
    @version（只出现在类和接口的文档中）
    @param（只出现在方法或构造器的文档中）
    @return（只出现在方法中）
    @exception（从java1.2之后也可以使用@thrown替代）
    @see
    @since
    @serial（也可以使用@serialField或@serialData替代）
    @deprecated
    ```


# 复杂注释样例

```java
/**
 * Graphics is the abstract base class for all graphics contexts
 * which allow an application to draw onto components realized on
 * various devices or onto off-screen images.
 * A Graphics object encapsulates the state information needed
 * for the various rendering operations that Java supports. This
 * state information includes:
 * <ul>
 * <li>The Component to draw on
 * <li>A translation origin for rendering and clipping coordinates
 * <li>The current clip
 * <li>The current color
 * <li>The current font
 * <li>The current logical pixel operation function (XOR or Paint)
 * <li>The current XOR alternation color
 * (see <a href="#setXORMode">setXORMode</a>)
 * </ul>
 * <p>
 * Coordinates are infinitely thin and lie between the pixels of the
 * output device.
 * Operations which draw the outline of a figure operate by traversing
 * along the infinitely thin path with a pixel-sized pen that hangs
 * down and to the right of the anchor point on the path.
 * Operations which fill a figure operate by filling the interior
 * of the infinitely thin path.
 * Operations which render horizontal text render the ascending
 * portion of the characters entirely above the baseline coordinate.
 * <p>
 * Some important points to consider are that drawing a figure that
 * covers a given rectangle will occupy one extra row of pixels on
 * the right and bottom edges compared to filling a figure that is
 * bounded by that same rectangle.
 * Also, drawing a horizontal line along the same y coordinate as
 * the baseline of a line of text will draw the line entirely below
 * the text except for any descenders.
 * Both of these properties are due to the pen hanging down and to
 * the right from the path that it traverses.
 * <p>
 * All coordinates which appear as arguments to the methods of this
 * Graphics object are considered relative to the translation origin
 * of this Graphics object prior to the invocation of the method.
 * All rendering operations modify only pixels which lie within the
 * area bounded by both the current clip of the graphics context
 * and the extents of the Component used to create the Graphics object.
 * 
 * @author Sami Shaio
 * @author Arthur van Hoff
 * @version %I%, %G%
 * @since 1.0
 */
public abstract class Graphics {

    /** 
     * Draws as much of the specified image as is currently available
     * with its northwest corner at the specified coordinate (x, y).
     * This method will return immediately in all cases, even if the
     * entire image has not yet been scaled, dithered and converted
     * for the current output device.
     * <p>
     * If the current output representation is not yet complete then
     * the method will return false and the indicated 
     * {@link ImageObserver} object will be notified as the
     * conversion process progresses.
     *
     * @param img the image to be drawn
     * @param x the x-coordinate of the northwest corner
     * of the destination rectangle in pixels
     * @param y the y-coordinate of the northwest corner
     * of the destination rectangle in pixels
     * @param observer the image observer to be notified as more
     * of the image is converted. May be 
     * <code>null</code>
     * @return <code>true</code> if the image is completely 
     * loaded and was painted successfully; 
     * <code>false</code> otherwise.
     * @see Image
     * @see ImageObserver
     * @since 1.0
     */
    public abstract boolean drawImage(Image img, int x, int y, 
                                      ImageObserver observer);


    /**
     * Dispose of the system resources used by this graphics context.
     * The Graphics context cannot be used after being disposed of.
     * While the finalization process of the garbage collector will
     * also dispose of the same system resources, due to the number
     * of Graphics objects that can be created in short time frames
     * it is preferable to manually free the associated resources
     * using this method rather than to rely on a finalization
     * process which may not happen for a long period of time.
     * <p>
     * Graphics objects which are provided as arguments to the paint
     * and update methods of Components are automatically disposed
     * by the system when those methods return. Programmers should,
     * for efficiency, call the dispose method when finished using
     * a Graphics object only if it was created directly from a
     * Component or another Graphics object.
     *
     * @see #create(int, int, int, int)
     * @see #finalize()
     * @see Component#getGraphics()
     * @see Component#paint(Graphics)
     * @see Component#update(Graphics)
     * @since 1.0
     */
    public abstract void dispose();

    /**
     * Disposes of this graphics context once it is no longer 
     * referenced.
     *
     * @see #dispose()
     * @since 1.0
     */
    public void finalize() {
        dispose();
    }
}
```