# Hello-Word
这个我的第一个github仓库
你好
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestHeader;
import org.springframework.web.bind.annotation.RestController;
import jakarta.servlet.http.HttpServletResponse;

import java.io.InputStream;
import java.net.HttpURLConnection;
import java.net.URL;

@RestController
public class ProxyController {

    @GetMapping("/proxy")
    public void proxy(
            @RequestHeader("X-Origin-Url") String originUrl,
            HttpServletResponse response
    ) throws Exception {
        System.out.println("目标地址：" + originUrl);

        if (!originUrl.startsWith("http://") && !originUrl.startsWith("https://")) {
            originUrl = "https://" + originUrl;
        }

        HttpURLConnection conn = (HttpURLConnection) new URL(originUrl).openConnection();
        conn.setRequestMethod("GET");
        conn.setConnectTimeout(5000);
        conn.setReadTimeout(10000);

        // 加自定义头
        conn.setRequestProperty("X-User-Id", "1001");
        conn.setRequestProperty("X-Token", "my-token-abc");

        // 透传响应状态码
        int status = conn.getResponseCode();
        response.setStatus(status);

        // 透传 Content-Type
        String contentType = conn.getContentType();
        if (contentType != null) {
            response.setContentType(contentType);
        }

        // 流式转发，不缓冲到内存，不会 OOM
        InputStream is = status >= 400 ? conn.getErrorStream() : conn.getInputStream();
        if (is != null) {
            try (is) {
                is.transferTo(response.getOutputStream());
            }
        }
    }
}
