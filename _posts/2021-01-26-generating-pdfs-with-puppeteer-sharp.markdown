---
layout: post
title:  "Generating PDFs with Puppeteer Sharp"
date:   2021-01-26 09:42:09 +0100
categories: pdf puppeteersharp
---

Recently I needed to generate PDFs in c#/.net from a console application.
There are many options to achieve this, but after some investigation I decided to use HTML as a template and generate the PDF from HTML.<br />
I had a few requirements for the library which would generate the PDF:

## Requirements
- ideally the library should be free
- it should display PNG images
- it should support custom headers / footers
- it should support manual page breaks
- it should be able show current page number / total pages
- (optional) the library should run on .net core

## Libraries comparison
I tried a couple of libraries, I'll described them below.

### SelectPdf
At first, I tried [SelectPdf][selectpdf-link].
It fullfills all of my requirements and was easy to use, the only thing which I did not like was, with the Community (free) version, there is a limitation - you can generate a PDF with 5 pages maximum.
There are multiple licenses in paid version, starting from from $499 / developer.

### PdfSharp
My other choice was [PdfSharp][pdfsharp-link].
This library <b> does not support .net Core</b>.
While doing some tests, I noticed some images were not rendered correctly or completely missing in the PDF.
This library does not support manual page breaks out of the box, altough I found a workaround on stackoverflow.com.
Also custom headers & footers are unsupported out of the box, with a possible workarounds.

### Wkhtmltopdf
I also found [WkHtmlToPdf][wkhtmltopdf-link], but I have not tried it, since this is a command line tool written in C, not a library. You can also find some c# wrappers  and give it a try.

### PuppeteerSharp
[Puppeteer Sharp][puppeteer-sharp-link] my final choice and I'll be describing this one a bit more.
As the authors stated, Puppeteer Sharp is a .NET port of the official Node.JS Puppeteer API - [Puppeteer][puppeteer-link]. It supports .NET Core.
Puppeteer is a Node library which provides a high-level API to control Chrome or Chromium.
Puppeteer runs headless by default, but can be configured to run full (non-headless).<br/>
It can be used for many things, e.g:
- taking a screenshot of a webpage
- automated testing
- creating PDF from a webpage


So let's see how we can convert HTML page to PDF using PuppeteerSharp.

<i>note: there will be some workarounds needed to fullfill all the requirements mentioned above</i>

Install the nuget package with:

{% highlight csharp %}
Install-Package PuppeteerSharp -Version 2.0.4
{% endhighlight %}

Sample code - generate PDF from HTML 

{% highlight csharp %}
var html = "<html>this is my awesome HTML content</html>";
var outputFile = @"output.pdf";
var browser = await Puppeteer.LaunchAsync();

using (var page = await browser.NewPageAsync())
{           
    await page.SetContentAsync(html);                                    
    await page.PdfAsync(outputFile);
}
{% endhighlight %}


By default a headless Chromium browser will be created.<br />
The first time your program is run, Chromium will be downloaded to a local folder (by default):
{% highlight csharp %}
.local-chromium
{% endhighlight %}

You can enable/disable <b>Headless</b> option, or set <b>ExecutablePath</b> if you have already installed Chrome on your system and want to use it instead of Chromium.

{% highlight csharp %}
var browser = await Puppeteer.LaunchAsync(new LaunchOptions
{
    Headless = true,
    ExecutablePath = @"c:\Program Files (x86)\Google\Chrome\Application\chrome.exe",
});
{% endhighlight %}
  
<br />
### Implementation
Let's see how can we implement all the required features.
<br />
<br />

#### Workaround - Displaying PNG images

Unfortunatelly, images defined in HTML as <b>&lt;img src="awesome-image.png" /&gt;</b> won't be displayed in the PDF document.
But we can convert the image to a base64 text with e.g:

{% highlight csharp %}
private static string ImageToBase64(string imagePath)
{
    using (Image image = Image.FromFile(imagePath))
    {
        using (MemoryStream m = new MemoryStream())
        {
            image.Save(m, image.RawFormat);
            byte[] imageBytes = m.ToArray();
            
            string base64String = Convert.ToBase64String(imageBytes);
            return base64String;
        }
    }
}
{% endhighlight %}

...and then set the <b>src=</b> like this:

{% highlight csharp %}
<img width="244" height="100" src="data:image/png;base64,/*base64 string here*/" />
{% endhighlight %}

<br />

#### Manual Page break
We can use CSS property <b>page-break-after</b>, then create a <b>.page</b> class in our HTML template page and then use it to display content on a separate page

{% highlight html %}
<style>
    .page {
        page-break-after: always;
    }
</style>	

<body>
    <div class="page">
        My awesome content!
        This will be displayed on the first page.
    </div>
    
    <div class="page">
        Even more awesome stuff here.
        Displayed on the second page.
    </div>
</body>
{% endhighlight %}

<br />

#### Headers / footers, display current page number / total pages count 

To enable a header or a footer, we need to set <b>DisplayHeaderFooter = true</b> and also define <b>HeaderTemplate</b> or <b>FooterTemplate</b>.
To keep it a bit more organized, I would recommend to create separate HTML templates for headers/footers, e.g. <b>header.html</b>, <b>footer.html</b>
and load html HTML content from them. For this example, I'll hardcode it as strings.

We can use also special CSS classes <b>pageNumber</b>, <b>totalPages</b> to display the current page number / total pages count.

{% highlight csharp %}
await page.PdfAsync(outputFile, new PdfOptions()
{
    DisplayHeaderFooter = true,
    MarginOptions = new MarginOptions()
    {
        // it's important to add some margin too
        Bottom = "40px",
        Top = "40px"
    },
    HeaderTemplate =
        "<div style=\"font-size:10px!important;color:grey!important;\" class=\"pdfheader\">" +
        "<span>Page: </span><span class=\"pageNumber\"></span> of <span class=\"totalPages\"></span></div>",
    FooterTemplate = "<div style=\"font-size:10px\">this will be a cool footer</div>"

});
{% endhighlight %}
 
<br />
<br />

### Complete code

Note: since the PuppeteerSharp api is <b>async</b>, we need to define <b>Main()</b> as <b>async Task</b> instead of <b>void</b>.

{% highlight csharp %}
using System;
using System.Drawing;
using System.IO;
using System.Text.RegularExpressions;
using System.Threading.Tasks;
using PuppeteerSharp;
using PuppeteerSharp.Media;

namespace PdfPuppeteerSharp
{
    class Program
    {    
        private static async Task Main()
        {            
            string fileName = @"template.html";
            string puppeteerSharpOutput = @"puppeteerSharp.pdf";
            string html = File.ReadAllText(fileName);
                       
            html = ProcessImages(html);
            await GeneratePdfPuppeteerSharp(html, puppeteerSharpOutput);
        }      

        private static string ProcessImages(string html)
        {            
            string pattern = "src=\"(?<src>[^\"]*)\"";
            string updatedHtml = html;

            foreach (var line in html.Split(Environment.NewLine))
            {
                var matches = Regex.Matches(line, pattern);

                if (matches.Count > 0)
                {
                    var path = matches[0].Groups["src"].Value;

                    string base64EncodedImage = ImageToBase64(path);
                    string base64ImageSource = @$"data:image/png;base64,{base64EncodedImage}";

                    updatedHtml = updatedHtml.Replace(path, base64ImageSource);
                }
            }

            return updatedHtml;
        }

        private static string ImageToBase64(string imagePath)
        {
            using (Image image = Image.FromFile(imagePath))
            {
                using (MemoryStream m = new MemoryStream())
                {
                    image.Save(m, image.RawFormat);
                    return Convert.ToBase64String(m.ToArray());
                }
            }
        }

        private static async Task GeneratePdfPuppeteerSharp(string html, string outputFile)
        {
            await new BrowserFetcher().DownloadAsync(BrowserFetcher.DefaultRevision);
            var browser = await Puppeteer.LaunchAsync(new LaunchOptions());

            using (var page = await browser.NewPageAsync())
            {
                await page.SetContentAsync(html);
                await page.PdfAsync(outputFile, new PdfOptions()
                {
                    DisplayHeaderFooter = true,
                    MarginOptions = new MarginOptions()
                    {
                        Bottom = "60px",
                        Top = "60px"
                    },
                    HeaderTemplate =
                        "<div style=\"font-size:10px!important;color:grey!important;margin:2px\" class=\"pdfheader\">" +
                        "<span>Page: </span><span class=\"pageNumber\"></span> of <span class=\"totalPages\"></span></div>",
                    FooterTemplate = "<div style=\"font-size:10px;margin:2px\">this will be your awesome footer</div>"
                });
            }
        }     
    }
}
{% endhighlight %}

<b>template.html</b> content:

{% highlight html %}
<html>
<head>
    <style>
    .page {				
            page-break-after: always;
        }
    </style>		
</head>	
<body>
    <div class="page">
        My awesome content!
        
        <img src="dealwithit.png" />
    </div>		
    <div class="page">
        Even more awesome stuff here
    </div>		
</body>
</html>
{% endhighlight %}


### Final words
If you want to build something more complex, probably you'll need a HTML templating engine, you can use e.g. Mustachio - [Mustachio][mustachio-link]


[puppeteer-sharp-link]: https://www.puppeteersharp.com/
[puppeteer-link]: https://github.com/puppeteer/puppeteer
[wkhtmltopdf-link]: https://wkhtmltopdf.org/
[pdfsharp-link]: http://www.pdfsharp.net/
[selectpdf-link]: https://selectpdf.com/
[mustachio-link]: https://github.com/wildbit/mustachio

