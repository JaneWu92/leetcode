Solution 1: No idea why invalid<br/>
```
import java.util.*;

public class Solution {
    public List<String> crawl(String startUrl, HtmlParser htmlParser) {
        int slashCount = 0;
        int hostSlashIndex = 0;
        for (int i = 0; i < startUrl.length(); i++) {
            if (startUrl.charAt(i) == '/') {
                slashCount++;
                if (slashCount == 3) {
                    hostSlashIndex = i;
                    break;
                }
            }
        }
        List<String> tempUrls = new ArrayList<String>();
        String host = startUrl.substring(0, hostSlashIndex);
        List<String> first = new ArrayList<String>();
        first.add(host);
        tempUrls.addAll(first);
        List<String> childs = childUrls(first, htmlParser);
        while (childs.size() != 0) {
            tempUrls.addAll(childs);
            childs = childUrls(childs, htmlParser);
        }
        List<String> resultUrls = new ArrayList<String>();
        for (String url : tempUrls) {
            if (url.contains(host)) {
                resultUrls.add(url);
            }
        }
        return resultUrls;
    }

    private List<String> childUrls(List<String> urls, HtmlParser htmlParser) {
        List<String> cUrls = new ArrayList<String>();
        for (String url : urls) {
            cUrls.addAll(htmlParser.getUrls(url));
        }
        return cUrls;
    }
}
```