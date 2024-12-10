
## 1、介绍


字体图标在Web应用中最为常见，字体图标是矢量的，矢量图意味着每个图标都能在所有大小的屏幕上完美呈现，可以随时更改大小和颜色，而且不失真。字体图标常见的有Font Awesome和Elegant Icon Font，她们不仅图标数量多，还可以免费使用。这些图标如果能用在WinForm项目中，不仅可以带来更加直观的界面效果，也能让图标不再局限于类似png类型，本文将介绍在WinForm项目中如何使用字体图标。


## 2、字体图标的选择


网上IconFont资源很多，同时很多提供SVG下载的网站都会提供对应的IconFont文件。本文就以：比较流行且开源免费的FontAwesome字体图标为例，讲解.NET开发的WinForm项目如何使用。


FontAwesome，官网：[https://fontawesome.com.cn/v4/icons](https://github.com)


![FontAwesome](https://img2024.cnblogs.com/blog/157572/202412/157572-20241209105158432-191834487.png)


在上图中，我们可以看到每个图标都有对应的Unicode编码，我们需要使用这个编码来做图标的展示。


## 3、使用方法


下载字体图标到本地，放到项目相应的位置，如在我们的项目中使用了两类字体图标，FontAwesome和ElegantIcon，如下图所示。


![字体图标](https://img2024.cnblogs.com/blog/157572/202412/157572-20241209105158402-1507171098.png)


![图标](https://img2024.cnblogs.com/blog/157572/202412/157572-20241209105158465-420438617.png)


在项目中定义字体编码对应的枚举部分代码如下所示。



```
/// 
/// 图标枚举,包含Awesome图标和Elegant图标，分别以A和E开头
/// 
public enum FontIcons 
{
    #region Awesome    English:Awesome
    /// 
    /// a fa 500PX
    /// 
    A_fa_500px = 0xf26e,
    /// 
    /// a fa address book
    /// 
    A_fa_address_book = 0xf2b9,
    /// 
    /// a fa address book o
    /// 
    A_fa_address_book_o = 0xf2ba,
    /// 
    /// a fa address card
    /// 
    A_fa_address_card = 0xf2bb,
    #endregion
    
    #region Elegant    English:Elegant
    /// 
    /// The e arrow up
    /// 
    E_arrow_up = 0x21,
    /// 
    /// The e arrow down
    /// 
    E_arrow_down = 0x22,
    /// 
    /// The e arrow left
    /// 
    E_arrow_left = 0x23,
    #endregion
}

```

定义字体图标加载公共类：FontImagesHelper.cs，此类不仅支持对待加载图标指定尺寸大小、还可以设置前景色和背景色。


![FontImagesHelper](https://img2024.cnblogs.com/blog/157572/202412/157572-20241209105158427-526590826.png)


FontImagesHelper.cs源码如下：



```
/// 
/// 字体图标图片,awesome字体默认加载，elegant字体在使用时延迟加载
/// 
public static class FontImagesHelper
{
    /// 
    /// The m font collection
    /// 
    private static readonly PrivateFontCollection fontCollection = new PrivateFontCollection();

    /// 
    /// The m fonts awesome
    /// 
    private static readonly Dictionary<string, Font> fontsAwesome = new Dictionary<string, Font>();

    /// 
    /// The m fonts elegant
    /// 
    private static readonly Dictionary<string, Font> fontsElegant = new Dictionary<string, Font>();

    /// 
    /// The m cache maximum size
    /// 
    private static Dictionary<int, float> cacheMaxSize = new Dictionary<int, float>();

    /// 
    /// The minimum font size
    /// 
    private const int MinFontSize = 8;

    /// 
    /// The maximum font size
    /// 
    private const int MaxFontSize = 43;

    /// 
    /// 构造函数
    /// 
    /// Font file not found
    static FontImagesHelper()
    {
        string strFile = Path.Combine(SystemInfo.StartupPath, "Resource", "IconFont\\FontAwesome.ttf");
        fontCollection.AddFontFile(strFile);
        float size = MinFontSize;
        for (int i = 0; i <= (MaxFontSize - MinFontSize) * 2; i++)
        {
            fontsAwesome.Add(size.ToString("F2"), new Font(fontCollection.Families[0], size, FontStyle.Regular, GraphicsUnit.Point));
            size += 0.5f;
        }
    }

    /// 
    /// 获取图标
    /// 
    /// 图标名称
    /// 图标大小
    /// 前景色
    /// 背景色
    /// 图标
    public static Icon GetIcon(string iconName, int imageSize = 32, Color? foreColor = null, Color? backColor = null)
    {
        FontIcons icon = (FontIcons)Enum.Parse(typeof(FontIcons), iconName);
        return GetIcon(icon, imageSize, foreColor, backColor);
    }

    /// 
    /// 获取图标
    /// 
    /// 图标名称
    /// 图标大小
    /// 前景色
    /// 背景色
    /// 图标
    public static Icon GetIcon(FontIcons iconName, int imageSize = 32, Color? foreColor = null, Color? backColor = null)
    {
        try
        {
            Bitmap image = GetImage(iconName, imageSize, foreColor, backColor);
            return image != null ? ToIcon(image, imageSize) : null;
        }
        catch
        {
            return null;
        }
    }

    /// 
    /// 获取图标
    /// 
    /// 图标名称
    /// 图标大小
    /// 前景色
    /// 背景色
    /// 图标
    public static Bitmap GetImage(string iconName, int imageSize = 32, Color? foreColor = null, Color? backColor = null)
    {
        try
        {
            FontIcons icon = (FontIcons)Enum.Parse(typeof(FontIcons), iconName);
            return GetImage(icon, imageSize, foreColor, backColor);
        }
        catch
        {
            return null;
        }
    }    

    /// 
    /// Gets the size of the icon.
    /// 
    /// The icon text.
    /// The graphics.
    /// The font.
    /// Size.
    private static Size GetIconSize(FontIcons iconName, Graphics graphics, Font font)
    {
        string text = char.ConvertFromUtf32((int)iconName);
        return graphics.MeasureString(text, font).ToSize();
    }

    /// 
    /// Converts to icon.
    /// 
    /// The source bitmap.
    /// The size.
    /// Icon.
    /// srcBitmap
    private static Icon ToIcon(Bitmap srcBitmap, int size)
    {
        if (srcBitmap == null)
        {
            throw new ArgumentNullException("srcBitmap");
        }

        Icon icon;
        using (MemoryStream memoryStream = new MemoryStream())
        {
            new Bitmap(srcBitmap, new Size(size, size)).Save(memoryStream, ImageFormat.Png);
            Stream stream = new MemoryStream();
            BinaryWriter binaryWriter = new BinaryWriter(stream);
            if (stream.Length <= 0L)
            {
                return null;
            }

            binaryWriter.Write((byte)0);
            binaryWriter.Write((byte)0);
            binaryWriter.Write((short)1);
            binaryWriter.Write((short)1);
            binaryWriter.Write((byte)size);
            binaryWriter.Write((byte)size);
            binaryWriter.Write((byte)0);
            binaryWriter.Write((byte)0);
            binaryWriter.Write((short)0);
            binaryWriter.Write((short)32);
            binaryWriter.Write((int)memoryStream.Length);
            binaryWriter.Write(22);
            binaryWriter.Write(memoryStream.ToArray());
            binaryWriter.Flush();
            binaryWriter.Seek(0, SeekOrigin.Begin);
            icon = new Icon(stream);
            stream.Dispose();
        }

        return icon;
    }
}

```

在RDIFramework.NET框架产品中整合了字体图标的使用，框架模块的图标按字体图标进行了整合加载，如下图所示。


![模块字体图标](https://img2024.cnblogs.com/blog/157572/202412/157572-20241209105158480-966360629.png)


调用对应图标的代码。



```
var img = FontImagesHelper.GetImage("A_fa_address_card", 26, "#66B9BF");

```

除了上面提到的字体图标，我们还可以使用阿里巴巴矢量图标库，地址：[https://www.iconfont.cn/](https://github.com)


## 4、参考文章


[iconfont\-阿里巴巴矢量图标库](https://github.com)


[FontAwesome 字体图标中文Icon](https://github.com):[悠兔机场](https://xinnongbo.com)


[RDIFramework.NET CS敏捷开发框架 V6\.1发布(.NET6\+、Framework双引擎、全网唯一)](https://github.com)


[.NET开发WinForm(C/S)项目整合三种服务访问模式(直连、WCF、WebAPI)](https://github.com)


