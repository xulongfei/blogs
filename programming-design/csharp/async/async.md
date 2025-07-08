## 异步编程

```csharp
using Demo.Http.Client;
using Kiwi.Matchmania;
using Google.Protobuf;

namespace Demo.Rank
{
    public sealed class HttpRankDataService: IRankDataService
    {
        private readonly UniversalHttpClient _httpClient;// HttpClient 实例 配置连接池参数？
        private  readonly string _baseHost;
        public HttpRankDataService(UniversalHttpClient httpClient, string baseHost = "http://127.0.0.1:9080")
        {
            _httpClient = httpClient;
            _baseHost = baseHost;
        }

        // 实现 IRankDataService 接口方法
        public async Task<UniRankRsp> GetRankData(UniRankReq req)
        {
            return await SendRequestAsync<UniRankReq, UniRankRsp>("/UniRankData", req);
        }

        // 实现 IRankDataService 接口方法
        public async Task<UniRankUpdateRsp> UniRankUpdate(UniRankUpdateReq req)
        {
            return await SendRequestAsync<UniRankUpdateReq, UniRankUpdateRsp>("/UniRankUpdate", req);
        }

        private string GetUrl(string path)
        {
            return $"{_baseHost}{path}";
        }

        /// <summary>
        /// 发送 JSON 请求并解析响应
        /// </summary>
        private async Task<TRsp> SendRequestAsync<TReq, TRsp>(string path, TReq req)
            where TReq : IMessage,new()
            where TRsp : IMessage,new()
        {
            string url = GetUrl(path);
            string json = JsonFormatter.Default.Format(req);
            Console.WriteLine($"Request to {url}: {json}");

            string resp = await _httpClient.PostJsonAsync(url, json, onError: (Exception ex) =>
            {
                Console.WriteLine($"Error: {ex.Message}");
            });
            return JsonParser.Default.Parse<TRsp>(resp);
        }
    }
}

```