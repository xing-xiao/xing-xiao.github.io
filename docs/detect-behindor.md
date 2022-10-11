# Maneo-Detect-Behinder
基于Flink的实时冰蝎(Behinder)流量检测

## 构建测试环境

基于kubernetes构建受害主机，[冰蝎Kubernetes环境构建](https://github.com/xing-xiao/Maneo-Detect-Behinder/tree/master/k8s-behinder-env)

pod中运行container分别为

- webshell: apache+php容器，提供webservice
- tcpdump: 抓取网络流量pcap包
- suricata、zeek: 抓取日志
- filebeat: 向SIEM logstash发送日志

## 行为分析

取到的日志如下

```
{"uuid":"hehindor-0x01","event_name":"bro-http","ts":"2019-09-26T08:52:50.998415Z","uid":"COCJ2z4rtgv6TLSVd7","src_ip":"192.168.10.1","src_port":51550,"dst_ip":"192.168.17.2","dst_port":8080,"trans_depth":1,"method":"GET","host":"192.168.17.2","uri":"/shell.php?pass=918","version":"1.1","user_agent":"Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/534.50 (KHTML, like Gecko) Version/5.1 Safari/534.50","request_body_len":0,"response_body_len":16,"status_code":200,"status_msg":"OK","tags":[],"resp_fuids":["FV0tp34bEkQzuIT3M8"],"resp_mime_types":["text/plain"],"header_host":"192.168.17.2:8080","header_accept":"text/html, image/gif, image/jpeg, *; q=.2, */*; q=.2","header_connection":"keep-alive","header_content_type":"application/x-www-form-urlencoded","server_header_names":["DATE","SERVER","X-POWERED-BY","SET-COOKIE","EXPIRES","CACHE-CONTROL","PRAGMA","CONTENT-LENGTH","KEEP-ALIVE","CONNECTION","CONTENT-TYPE"],"server_header_values":["Thu, 26 Sep 2019 08:50:31 GMT","Apache/2.4.25 (Debian)","PHP/7.0.33","PHPSESSID=a3d51cad592f042f3bc2f4d0d88ce015; path=/","Thu, 19 Nov 1981 08:52:00 GMT","no-store, no-cache, must-revalidate","no-cache","16","timeout=5, max=100","Keep-Alive","text/html; charset=UTF-8"],"body":"121f28cf6172a20a","http_response_time":0.001194}
{"uuid":"hehindor-0x02","event_name":"bro-http","ts":"2019-09-26T08:52:51.008729Z","uid":"COCJ2z4rtgv6TLSVd7","src_ip":"192.168.10.1","src_port":51550,"dst_ip":"192.168.17.2","dst_port":8080,"trans_depth":2,"method":"GET","host":"192.168.17.2","uri":"/shell.php?pass=230","version":"1.1","user_agent":"Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/534.50 (KHTML, like Gecko) Version/5.1 Safari/534.50","request_body_len":0,"response_body_len":16,"status_code":200,"status_msg":"OK","tags":[],"resp_fuids":["FUTA6c80gCk8iHm8j"],"resp_mime_types":["text/plain"],"header_host":"192.168.17.2:8080","header_accept":"text/html, image/gif, image/jpeg, *; q=.2, */*; q=.2","header_connection":"keep-alive","header_content_type":"application/x-www-form-urlencoded","server_header_names":["DATE","SERVER","X-POWERED-BY","SET-COOKIE","EXPIRES","CACHE-CONTROL","PRAGMA","CONTENT-LENGTH","KEEP-ALIVE","CONNECTION","CONTENT-TYPE"],"server_header_values":["Thu, 26 Sep 2019 08:50:31 GMT","Apache/2.4.25 (Debian)","PHP/7.0.33","PHPSESSID=0974cb5601ef4eaa3c15081b50c75179; path=/","Thu, 19 Nov 1981 08:52:00 GMT","no-store, no-cache, must-revalidate","no-cache","16","timeout=5, max=99","Keep-Alive","text/html; charset=UTF-8"],"body":"39ac0864a89c5c01","http_response_time":0.000804}
{"uuid":"hehindor-0x03","event_name":"bro-http","ts":"2019-09-26T08:52:51.011900Z","uid":"COCJ2z4rtgv6TLSVd7","src_ip":"192.168.10.1","src_port":51550,"dst_ip":"192.168.17.2","dst_port":8080,"trans_depth":3,"method":"POST","host":"192.168.17.2","uri":"/shell.php","version":"1.1","user_agent":"Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/534.50 (KHTML, like Gecko) Version/5.1 Safari/534.50","request_body_len":1112,"response_body_len":128,"status_code":200,"status_msg":"OK","tags":[],"orig_fuids":["Fcop9A1viCj9JThXn8"],"orig_mime_types":["text/plain"],"resp_fuids":["FSt7FV3XCRGGCkTbLa"],"resp_mime_types":["text/plain"],"post_body":"5jwfZRdoTznxAGNImafH3S5tFnXRDpj3+1kiFRFw4mzKhi3umkKWpXbKUgJRKegemv0uvF36rgdKutUVhtnMUW9CinNuOoC4l4n0xoAJfFNlTuxRPZ9/lobBY4BwzFpX4q2kW33RMCHhpNukeJ24hsWoxW+pCI+dQBYS2meszBOz1xWmioYAl8YGG2+p8wDXKZlu78sD...5jwfZRdoTznxAGNImafH3S5tFnXRDpj3+1kiFRFw4mzKhi3umkKWpXbKUgJRKegemv0uvF36rgdKutUVhtnMUW9CinNuOoC4l4n0xoAJfFNlTuxRPZ9/lobBY4BwzFpX4q2kW33RMCHhpNukeJ24hsWoxW+pCI+dQBYS2meszBOz1xWmioYAl8YGG2+p8wDXKZlu78sDU9JTvY0jaatkSP2oRQgOinou3Bzy9+l7584Wh0JArnwInFc4dmqWQLsba58a1odG7ruWHHLaVwRfskZ6KaYCQ+OPcgCm2kk8tb/VlRZlmqgTN08vNchRWh26kdIdifJc9qdDZMwE0hHNiz3Taof2g7e4g/7bFt+UDt4hNPfTfNGNACdKw9PwgL5nNsrzwR8CBxPGheIk+JtibirUw+9pCEpqLaelXjsdYJ5m0GY0RqmcCZwNKanJd1rHCIbdV+OYFNPP4GGK7kz2ZFkPRQLblz1FR0JtzH7A6xbWeppPx65ABuj3Jg7cSVcegUxVTK9eWyD4Sw45d5l6atET1m1El2LCWejGc4arw65EGoYEEzimQTXDrRZ/OBCv5LEYkLDwxoC17sEd6u6hoZhyz6M/efeq/+aqCIXei2GO3BRPrMeCBYS4NfXRZDidNcWL4vcjpxmoFN+RenaX7LhvAFPrLbbpyjWVp2cbkWP3ma2m9ap9JTqayOtSVZzupaMqqvRCyuMu+03R31Hopy1FDXvJnvjGsL46643ezUP7e8+1j7yTRAiKWc7QVEW7zaQwj3sC1fJ0ZUjXk0Lpr7mviEnxAtE+ZGrEnAGjUhpkh2zbsMiiircnl2xsD4If1zy49r5swGiEY2RnaoTA1H5FE5PvXcRvk2vntdBK52W8xB0PHa1nXHLdewmScPRLR0zaXxxizfES1xZw3HsUDypAjsjcZLBmiTISxKrH5JM03qpO/lgjGTyrNzbgpv+yPuf2mt6rY25zJWmHSutnAT9Q1JbZwc86rPRHbHoN33/zcvZ5ObBRxXqZ4GVW64USIUFed90BAWTpJpIkAbfv8S/y53PBO22aM4dtUAWon5VzYp0qgM5XOz7VzTbgrRfAf+obxFTEnxXCB6v0Bqp1OA==","header_host":"192.168.17.2:8080","header_accept":"text/html, image/gif, image/jpeg, *; q=.2, */*; q=.2","header_connection":"keep-alive","header_cookie":"PHPSESSID=0974cb5601ef4eaa3c15081b50c75179; path=/","header_content_length":"1112","header_content_type":"application/x-www-form-urlencoded","server_header_names":["DATE","SERVER","X-POWERED-BY","EXPIRES","CACHE-CONTROL","PRAGMA","VARY","CONTENT-LENGTH","KEEP-ALIVE","CONNECTION","CONTENT-TYPE"],"server_header_values":["Thu, 26 Sep 2019 08:50:31 GMT","Apache/2.4.25 (Debian)","PHP/7.0.33","Thu, 19 Nov 1981 08:52:00 GMT","no-store, no-cache, must-revalidate","no-cache","Accept-Encoding","128","timeout=5, max=98","Keep-Alive","text/html; charset=UTF-8"],"body":"P9EFB41x+3gdZi9HJ8iyf5sXkOkcuCuIPA1wxvghxbzH99SH+jvS+yl6fvHH19WOufx5+BA9b7E4x4g3UBhLlgxLhSuUPCKK83aj9Gzzl91HFx8/Ulhy/GOLmV+xTJzE","http_response_time":0.000791,"cookie_vars":["PHPSESSID","path"]}
{"uuid":"hehindor-0x04","event_name":"bro-http","ts":"2019-09-26T08:52:51.015303Z","uid":"COCJ2z4rtgv6TLSVd7","src_ip":"192.168.10.1","src_port":51550,"dst_ip":"192.168.17.2","dst_port":8080,"trans_depth":4,"method":"POST","host":"192.168.17.2","uri":"/shell.php","version":"1.1","user_agent":"Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/534.50 (KHTML, like Gecko) Version/5.1 Safari/534.50","request_body_len":2220,"response_body_len":160128,"status_code":200,"status_msg":"OK","tags":[],"orig_fuids":["FJtxT62pun6hhMFSk5"],"orig_mime_types":["text/plain"],"resp_fuids":["FINOwu2IADKnzzPCti"],"resp_mime_types":["text/plain"],"post_body":"5jwfZRdoTznxAGNImafH3TuwKogooJYGEdn/prs1SpZQk6fHaXguLDrhjpND5am7OudYViWzKNPZzzaZSDH8hS+VXJqPMCdR7wKG3upwDME4qhJISt24UImMIiavajImQLhMwWhGpbR4DEdwgYBhXlegjUE4MaRCNmcBVBeP3WiUq+QqqqtxMUxGWjt5/eEcgWGCHKKd...5jwfZRdoTznxAGNImafH3TuwKogooJYGEdn/prs1SpZQk6fHaXguLDrhjpND5am7OudYViWzKNPZzzaZSDH8hS+VXJqPMCdR7wKG3upwDME4qhJISt24UImMIiavajImQLhMwWhGpbR4DEdwgYBhXlegjUE4MaRCNmcBVBeP3WiUq+QqqqtxMUxGWjt5/eEcgWGCHKKdxNDeJUeIReiSR5EHXNfAntbYQ6/ad5r8+Ev5wyn4kb1tex7NqANSn5N+Ime6M8+gGve1ap4iBUnSAC9kG/hfV/bWcShjbZZmnyOqxD7TzCIV0HWP4YP+aodNeyiXFXwYJUX7TmVM1R0oBGvmyuDUov4Mdij6xPdji4mHqDHloiqrAHN8W3qunK81WJ2qdB/6Gmou8ppTHalU6r6JGpHNyEAIehKZp7Vv3KHXZ3o3uv5hIRJ6HSV6v/C8AsbFBPkHTdlSAkAlqGeG8KVsxhQ78vk/kJNys9Kx3s2AFlhN5+MypEvK3AUCqexBMqsHk4kh6c9XYbm4uCENhscFvZM3yiSJdmpSdE5Q2tpMks64867/gYWF96bmScAan9XdQfdtlL6EL8FEpSI5yoK1dpFd6jRGMzQcEfaGRBCwJxvTlZutbzL2+Oryd8AGeEbFcYyQ717h/gKBj16koh8yb5H/Edruk89fI1T57wcSAMUi49WHb6HRfZ8VtTVyT647QXuOvJPp9H0CoT896itaA2/P8xEJqvJ6V1tkqbsYE9dEO97oqTXuM0AoMcwC9mx5E4jHBWBl89FzPSG1MRLpGilokJ/RjfxYnxcR9g9CYJI6/9ebQ7UaythC7+Iyd90LSBDpYp61uHzpBGlMph1LdsFKcYNwq3SXV3yGQuC11vZISAS19cfOjhBXJ6hlqjD5gfdM4FNoR/oz+p8ejw8Jlr5GEfboYbUHx7y0P43U5AzQvdgHrfqfZNVllHQA1U4OXixTcd06wJOmIrxvHnilLoTEcrVlK+EvoDO6SuzBrUlLgUzrgJMcXxrl83i2xVXPg3G0+G3sc4r4CPXcQrme70w2Mn3wzUHs4x44vIXwKFS1/XFEBxZsKRvbKnBcfz9u01jLoocezgfYYYeLUuc9XVIr4t+2QCO0ywnrBHFxjOnr/F/lpXWSo26Q+m50/TJGtmfC89jvgzI39Rr8iPcipOgx6sxB05mBe9ojP3kQN6X/yPg1kHVejtDvIIxvYz+kIMDEcNocp+NRCeFc7pFP3XNipk4sQMMGtZn0NNmizhcLLV7HtAcB/SSi0QiZomwk+fF7cj79weo23HMPWgjWfbhiEO4bLHUE2jRK2E5qOYa3XtKBcgDgLikoq6f54uueb9U1sASUyBRDK2Eoqc8tQWQP7YbbqmzTP2qCqv0XpXqWhEeo00Pd8vF00aZlaOZ2o/FF6gn8U+JqdrFx7X2/T7yDr1ImRcUVHg3oaT/O2ZmUKbUeb7cDEdgCE4/iOJ+xiVrMelPrtdvtJQ5uqnEEJfQsCG0tNasYOpv1Msp0rxK8pXYTWPdsmtttzEe9BkcTwOKoz/9gXKEAu2hUv32vb+EBUcxQ4JJPPXt9AaYgrX05nsJuUL63Xox3UDwYrt+7ZYv6oRJy6sgJ/yMKm/QjP4he7tk2gyK+3fos8kFRD4h6hO9vfdRKnK0yUdgUTYYCB09wrVhht1wHdExYNSCMFxcY9T75t71iD9X4YSB6tzEJIvR4wXYlevJJ+ziyUr6uDSuob15z4Cu6hYi+7NRb73NnG31NYlLFKjV2FmzW7Km++tPIgiZKV+tm7Rklb/Vuj...","header_host":"192.168.17.2:8080","header_accept":"text/html, image/gif, image/jpeg, *; q=.2, */*; q=.2","header_connection":"keep-alive","header_cookie":"PHPSESSID=0974cb5601ef4eaa3c15081b50c75179; path=/","header_content_length":"2220","header_content_type":"application/x-www-form-urlencoded","server_header_names":["DATE","SERVER","X-POWERED-BY","EXPIRES","CACHE-CONTROL","PRAGMA","VARY","KEEP-ALIVE","CONNECTION","TRANSFER-ENCODING","CONTENT-TYPE"],"server_header_values":["Thu, 26 Sep 2019 08:50:31 GMT","Apache/2.4.25 (Debian)","PHP/7.0.33","Thu, 19 Nov 1981 08:52:00 GMT","no-store, no-cache, must-revalidate","no-cache","Accept-Encoding","timeout=5, max=97","Keep-Alive","chunked","text/html; charset=UTF-8"],"body":"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...","http_response_time":0.019064,"cookie_vars":["PHPSESSID","path"]}
{"uuid":"hehindor-0x05","event_name":"bro-http","ts":"2019-09-26T09:00:51.368468Z","uid":"CtxFYy1uNt30Gntaz5","src_ip":"192.168.10.1","src_port":52254,"dst_ip":"192.168.17.2","dst_port":8080,"trans_depth":1,"method":"POST","host":"192.168.17.2","uri":"/shell.php","version":"1.1","user_agent":"Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/534.50 (KHTML, like Gecko) Version/5.1 Safari/534.50","request_body_len":2220,"response_body_len":160128,"status_code":200,"status_msg":"OK","tags":[],"orig_fuids":["Fws2sG1fqvooRQTZ3j"],"orig_mime_types":["text/plain"],"resp_fuids":["FF7PVu1V5nNsRMVEzd"],"resp_mime_types":["text/plain"],"post_body":"5jwfZRdoTznxAGNImafH3TuwKogooJYGEdn/prs1SpZQk6fHaXguLDrhjpND5am7OudYViWzKNPZzzaZSDH8hS+VXJqPMCdR7wKG3upwDME4qhJISt24UImMIiavajImQLhMwWhGpbR4DEdwgYBhXlegjUE4MaRCNmcBVBeP3WiUq+QqqqtxMUxGWjt5/eEcgWGCHKKd...5jwfZRdoTznxAGNImafH3TuwKogooJYGEdn/prs1SpZQk6fHaXguLDrhjpND5am7OudYViWzKNPZzzaZSDH8hS+VXJqPMCdR7wKG3upwDME4qhJISt24UImMIiavajImQLhMwWhGpbR4DEdwgYBhXlegjUE4MaRCNmcBVBeP3WiUq+QqqqtxMUxGWjt5/eEcgWGCHKKdxNDeJUeIReiSR5EHXNfAntbYQ6/ad5r8+Ev5wyn4kb1tex7NqANSn5N+Ime6M8+gGve1ap4iBUnSAC9kG/hfV/bWcShjbZZmnyOqxD7TzCIV0HWP4YP+aodNeyiXFXwYJUX7TmVM1R0oBGvmyuDUov4Mdij6xPdji4mHqDHloiqrAHN8W3qunK81WJ2qdB/6Gmou8ppTHalU6r6JGpHNyEAIehKZp7Vv3KHXZ3o3uv5hIRJ6HSV6v/C8AsbFBPkHTdlSAkAlqGeG8KVsxhQ78vk/kJNys9Kx3s2AFlhN5+MypEvK3AUCqexBMqsHk4kh6c9XYbm4uCENhscFvZM3yiSJdmpSdE5Q2tpMks64867/gYWF96bmScAan9XdQfdtlL6EL8FEpSI5yoK1dpFd6jRGMzQcEfaGRBCwJxvTlZutbzL2+Oryd8AGeEbFcYyQ717h/gKBj16koh8yb5H/Edruk89fI1T57wcSAMUi49WHb6HRfZ8VtTVyT647QXuOvJPp9H0CoT896itaA2/P8xEJqvJ6V1tkqbsYE9dEO97oqTXuM0AoMcwC9mx5E4jHBWBl89FzPSG1MRLpGilokJ/RjfxYnxcR9g9CYJI6/9ebQ7UaythC7+Iyd90LSBDpYp61uHzpBGlMph1LdsFKcYNwq3SXV3yGQuC11vZISAS19cfOjhBXJ6hlqjD5gfdM4FNoR/oz+p8ejw8Jlr5GEfboYbUHx7y0P43U5AzQvdgHrfqfZNVllHQA1U4OXixTcd06wJOmIrxvHnilLoTEcrVlK+EvoDO6SuzBrUlLgUzrgJMcXxrl83i2xVXPg3G0+G3sc4r4CPXcQrme70w2Mn3wzUHs4x44vIXwKFS1/XFEBxZsKRvbKnBcfz9u01jLoocezgfYYYeLUuc9XVIr4t+2QCO0ywnrBHFxjOnr/F/lpXWSo26Q+m50/TJGtmfC89jvgzI39Rr8iPcipOgx6sxB05mBe9ojP3kQN6X/yPg1kHVejtDvIIxvYz+kIMDEcNocp+NRCeFc7pFP3XNipk4sQMMGtZn0NNmizhcLLV7HtAcB/SSi0QiZomwk+fF7cj79weo23HMPWgjWfbhiEO4bLHUE2jRK2E5qOYa3XtKBcgDgLikoq6f54uueb9U1sASUyBRDK2Eoqc8tQWQP7YbbqmzTP2qCqv0XpXqWhEeo00Pd8vF00aZlaOZ2o/FF6gn8U+JqdrFx7X2/T7yDr1ImRcUVHg3oaT/O2ZmUKbUeb7cDEdgCE4/iOJ+xiVrMelPrtdvtJQ5uqnEEJfQsCG0tNasYOpv1Msp0rxK8pXYTWPdsmtttzEe9BkcTwOKoz/9gXKEAu2hUv32vb+EBUcxQ4JJPPXt9AaYgrX05nsJuUL63Xox3UDwYrt+7ZYv6oRJy6sgJ/yMKm/QjP4he7tk2gyK+3fos8kFRD4h6hO9vfdRKnK0yUdgUTYYCB09wrVhht1wHdExYNSCMFxcY9T75t71iD9X4YSB6tzEJIvR4wXYlevJJ+ziyUr6uDSuob15z4Cu6hYi+7NRb73NnG31NYlLFKjV2FmzW7Km++tPIgiZKV+tm7Rklb/Vuj...","header_host":"192.168.17.2:8080","header_accept":"text/html, image/gif, image/jpeg, *; q=.2, */*; q=.2","header_connection":"keep-alive","header_cookie":"PHPSESSID=0974cb5601ef4eaa3c15081b50c75179; path=/","header_content_length":"2220","header_content_type":"application/x-www-form-urlencoded","server_header_names":["DATE","SERVER","X-POWERED-BY","EXPIRES","CACHE-CONTROL","PRAGMA","VARY","KEEP-ALIVE","CONNECTION","TRANSFER-ENCODING","CONTENT-TYPE"],"server_header_values":["Thu, 26 Sep 2019 08:58:32 GMT","Apache/2.4.25 (Debian)","PHP/7.0.33","Thu, 19 Nov 1981 08:52:00 GMT","no-store, no-cache, must-revalidate","no-cache","Accept-Encoding","timeout=5, max=100","Keep-Alive","chunked","text/html; charset=UTF-8"],"body":"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...","http_response_time":0.020398,"cookie_vars":["PHPSESSID","path"]}
```

特征如下

客户端会先和服务端进行两次密钥交换，生成16位长的密钥，然后通过密钥加密后续内容，使用base64编码填入post请求和response回包。所以的检测思路如下。

- Pattern1: `GET /uri?pwd=NUMBER`两遍，`CONTENT-TYPE: application/x-www-form-urlencoded`， 返回包长度位16。
- Pattern2: `POST /uri`， `CONTENT-TYPE: application/x-www-form-urlencoded`，post包内容是base64编码字符串，返回包体是base64编码字符串。
- Pattern: ` Pattern.begin[HttpEvent]("start").where(_.pattern == 1).next("middle").where(_.pattern == 1).followedBy("end").where(_.pattern == 2).within(Time.seconds(10))`


## Flink Job

根据上述规则，构造Flink Job检测冰蝎。[Flink检测冰蝎代码](https://github.com/xing-xiao/Maneo-Detect-Behinder/blob/master/flink-behinder-detector/src/main/scala/com/maneo/Behinder/behinder.scala)

```scala
case class HttpEvent(src_ip: String, dst_ip: String, dst_port: Int, uri: String, pattern: Int, uuid: String)

object HttpString {
  def unapply(str: String): Option[HttpEvent] = {
    try {
      val j: JSONObject = JSON.parseObject(str)
      if (j.getString("event_name") != "bro-http" || j.getString("header_content_type") != "application/x-www-form-urlencoded") {
        return None
      }
      val patternUri = ".*\\?((?!&).)*=\\d+$"
      val patternBase64 = "^(([A-Za-z0-9+/]{4})+[A-Za-z0-9+/\\.]{0,6})+([A-Za-z0-9+/]{3}=|[A-Za-z0-9+/]{2}==)?$"
      j.getString("method") match {
        case m: String if m.toUpperCase() == "GET" && j.getIntValue("response_body_len") == 16 && j.getString("uri").matches(patternUri) => Some(HttpEvent(j.getString("src_ip"), j.getString("dst_ip"), j.getIntValue("dst_port"), j.getString("uri").split("\\?")(0), 1, j.getString("uuid")))
        case m: String if m.toUpperCase() == "POST" && !j.getString("uri").contains("?") && j.getString("post_body").matches(patternBase64) && j.getString("body").matches(patternBase64) => Some(HttpEvent(j.getString("src_ip"), j.getString("dst_ip"), j.getIntValue("dst_port"), j.getString("uri"), 2, j.getString("uuid")))
        case _ => None
      }
    } catch { case _: Exception => None}
  }
}

object behinder {
  def main(args: Array[String]): Unit = {

    val eventStream: DataStream[HttpEvent] = env
      .kafkaSource.map(i => HttpString.unapply(i))
      .filter(_.nonEmpty)
      .map(i => i.get)

    val partitionedInput = eventStream.keyBy("uri", "src_ip", "dst_ip", "dst_port")

    val pattern: Pattern[HttpEvent, HttpEvent] = Pattern.begin[HttpEvent]("start").where(_.pattern == 1)
        .next("middle").where(_.pattern == 1)
        .followedBy("end").where(_.pattern == 2)
      .within(Time.seconds(10))

    val patternStream: PatternStream[HttpEvent] = CEP.pattern(partitionedInput, pattern)

    val alertStream = patternStream.select(
      (pattern: Map[String, Iterable[HttpEvent]]) => {
        val start: HttpEvent = pattern.getOrElse("start", null).iterator.next()
        val middle: HttpEvent = pattern.getOrElse("middle", null).iterator.next()
        val end: HttpEvent = pattern.getOrElse("end", null).iterator.next()
        (start.uri, start.pattern, middle.pattern, end.pattern)
      }
    )

    alertStream.print()

  }
}
```

## Reference

- [“冰蝎”动态二进制加密网站管理客户端 - Github](https://github.com/rebeyond/Behinder/releases)
- [冰蝎动态二进制加密WebShell特征分析 - Freebuf](https://www.freebuf.com/articles/web/213905.html)