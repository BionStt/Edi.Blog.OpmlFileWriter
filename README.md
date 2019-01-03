# Edi.Blog.OpmlFileWriter
OPML File writer used in my blog (ASP.NET Core)

[![Build status](https://dev.azure.com/ediwang/EdiBlog/_apis/build/status/EdiBlog-.NET%20Desktop-CI)](https://dev.azure.com/ediwang/EdiBlog/_build/latest?definitionId=36)

## Usage

Install via NuGet

```
Install-Package Edi.Blog.OpmlFileWriter
```

Example Code

```
public async Task<IActionResult> Index()
{
    var opmlDataFile = $"{AppDomain.CurrentDomain.GetData(Constants.DataDirectory)}\\opml.xml";
    if (!System.IO.File.Exists(opmlDataFile))
    {
        Logger.LogInformation($"OPML file not found, writing new file on {opmlDataFile}");

        var catInfos = _categoryService.GetAllCategories().Select(c => new OpmlCategoryInfo()
        {
            DisplayName = c.DisplayName,
            Title = c.Title
        });

        var oi = new OpmlInfo
        {
            SiteTitle = $"{AppSettings.SiteTitle} - OPML",
            CategoryInfo = catInfos,
            HtmlUrl = $"{HttpContext.Request.Scheme}://{HttpContext.Request.Host}/post",
            XmlUrl = $"{HttpContext.Request.Scheme}://{HttpContext.Request.Host}/rss",
            CategoryXmlUrlTemplate = $"{HttpContext.Request.Scheme}://{HttpContext.Request.Host}/rss/category/[catTitle]",
            CategoryHtmlUrlTemplate = $"{HttpContext.Request.Scheme}://{HttpContext.Request.Host}/category/list/[catTitle]"
        };

        await FileSystemOpmlWriter.WriteOpmlFileAsync($"{AppDomain.CurrentDomain.GetData(Constants.DataDirectory)}\\opml.xml", oi);
        Logger.LogInformation($"OPML file write completed.");

        if (!System.IO.File.Exists(opmlDataFile))
        {
            Logger.LogInformation($"OPML file still not found, something just went very very wrong...");
            return NotFound();
        }
    }

    string opmlContent = await Utils.ReadTextAsync(opmlDataFile, Encoding.UTF8);
    if (opmlContent.Length > 0)
    {
        return Content(opmlContent, "text/xml");
    }

    return NotFound();
}
```
