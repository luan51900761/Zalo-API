# Zalo-API
Send message to user from OA account using c# 
using System;
using System.Net.Http.Headers;
using System.Net.Http;
using System.Text.Json;
using System.Text;
using System.Threading.Tasks;
using System.Collections.Generic;

namespace zalo
{
    internal class Program
    {

        public class ZaloApiService
        {
            private readonly HttpClient _httpClient;
            private string accessToken;
            private readonly string _secretKey;
            private const string BaseUrl = "https://openapi.zalo.me/v3.0/oa/message/cs";

            public ZaloApiService(string secretKey, string appId, string refreshToken)
            {
                _httpClient = new HttpClient();
                _httpClient.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
                _secretKey = secretKey;
                accessToken = getAccessToken(secretKey, appId: appId, refreshToken: refreshToken);
            }
            public string getAccessToken(string secretKey, string appId, string refreshToken)
            {
                _httpClient.DefaultRequestHeaders.Add("Secret_key", secretKey);
                var requestBody = new FormUrlEncodedContent(new[]
                {
                   new KeyValuePair<string, string>("app_id", appId),
                   new KeyValuePair<string, string>("grant_type", "refresh_token"),
                   new KeyValuePair<string, string>("refresh_token", refreshToken)
               });
                var response = _httpClient.PostAsync("https://oauth.zaloapp.com/v4/oa/access_token", requestBody).Result;
                var responseContent = response.Content.ReadAsStringAsync().Result;

                if (response.IsSuccessStatusCode)
                {
                    var jsonResponse = JsonSerializer.Deserialize<Dictionary<string, string>>(responseContent);
                    return jsonResponse != null && jsonResponse.ContainsKey("access_token") ? jsonResponse["access_token"] : null;
                }
                else
                {
                    Console.WriteLine($"Error fetching access token. Status code: {response.StatusCode}, Content: {responseContent}");
                    return null;
                }
            }

            public async Task<bool> SendTextMessageAsync(string recipientId, string message)
            {
                var requestData = new
                {
                    recipient = new { user_id = recipientId },
                    message = new { text = message }
                };

                return await PostAsync($"{BaseUrl}", requestData);
            }

            private async Task<bool> PostAsync<T>(string endpoint, T data)
            {
                try
                {
                    var jsonRequest = JsonSerializer.Serialize(data);
                    var content = new StringContent(jsonRequest, Encoding.UTF8, "application/json");

                    content.Headers.Add("access_token", accessToken);

                    var response = await _httpClient.PostAsync(endpoint, content);
                    return await HandleResponseAsync(response);

                }
                catch (Exception ex)
                {
                    Console.WriteLine($"Error: {ex.Message}");
                    return false;
                }
            }

            private async Task<bool> HandleResponseAsync(HttpResponseMessage response)
            {
                var responseContent = await response.Content.ReadAsStringAsync();

                if (response.IsSuccessStatusCode)
                {
                    return true;
                }
                else
                {
                    Console.WriteLine($"Error sending Zalo message. Status code: {response.StatusCode}, Content: {responseContent}");
                    return false;
                }
            }
        }

        static async Task Main(string[] args)
        {
            //check accces token
            // if access token is valid then send message
            // if not valid reCreate accesstolen from refresh token
            ZaloApiService zl = new ZaloApiService("3813P8L18B8WL4neUN11", "4002546623234778204", "7Uo6SQ3yOrGRw85sfFbbK1xZXt6dxmykP8ljMARVTJjXbU1hbk956swJss-hoMOqJfpZLvJiH0WkdgPymiX4Mo2Zt7EPjY4iHD2rLeQA5G1pt8D8a9aAQch8aWs6g0rGJT-93l2ADbWk-AWnnfGEUGBPartOhYO0ElgYU--IIWaauiv-vO5j7p2IXLhr-I1zFhgtMEJi3dyifv4xvl9p72tayaAHWbCWKvFHS9NYTr5Tf_DplUnxKN7t_WA5v7qXSAxaOx7hKGD9WCTxbD1rCY-TbaxXsITeNfUxF825BtHtyhSkofW_Tp38g0_GZ1PR1VglFiwsI7aWeySSrU0aI0kAh3NBa2yT9EADEzcfEsaRsB8AkxCp8WJxYqVXXXaN6lJmRiEnQHu4elfXPaoD8j0HfE1YK0");
            await zl.SendTextMessageAsync("2020830992532917467", "Chúc bạn một buổi tối tốt lành!");
            Console.WriteLine("Message sent!");
        }
    }
}

//check acces token
//curl ^ "https://developers.zalo.me/api/apps/get-token-info^" ^
//  -H ^ "accept: */*^" ^
//  -H ^ "accept-language: vi-VN,vi;q=0.9,fr-FR;q=0.8,fr;q=0.7,en-US;q=0.6,en;q=0.5,ja;q=0.4^" ^
//  -H ^ "content-type: application/x-www-form-urlencoded^" ^
//  -b ^ "__zi=3000.SSZzejyD6zOgdh2mtnLQWYQN_RAG01ICFjIXe9fEM8qtdkgYbabTY7wVxgFJIbgBSPNleJ4u.1; __zi-legacy=3000.SSZzejyD6zOgdh2mtnLQWYQN_RAG01ICFjIXe9fEM8qtdkgYbabTY7wVxgFJIbgBSPNleJ4u.1; zpsid=33fQ.197735026.15.7lMAc4Uu6l8rYjFvIxWQyZBBQCD-Zot1SeWen4a3VQsRK5lDHl7b6I2u6l8; zoaw_sek=N_YC.783384777.2.FW3HNeVbsgzEPMk1X-NQhuVbsg-7vTglXWXgGU3bsgy; zoaw_type=0; _gcl_au=1.1.1688636309.1744373786; _fbp=fb.1.1744373787050.11219018756656247; developer_sid=olgUGQT0Q7FJ-996eaLpSjJWYnlQAYSgpDJUGxSjI2ooqCvHfWfu3gV3vaYWPNCYbz_lGeCnJdEXYTSdaL1J5xwVr7xATYqokuZKMy0WVplrpjnHnY1X2tDfP0; _gid=GA1.2.1637949099.1744634338; _zlang=vn; googtrans=/auto/vi; googtrans=/auto/vi; _ga_QKW3RCKZFH=GS1.1.1744637148.2.1.1744637389.0.0.0; _ga_NVN38N77J3=GS1.2.1744644980.7.1.1744646515.0.0.0; _ga_E63JS7SPBL=GS1.1.1744644979.7.1.1744646520.54.0.0; _ga_WSPJQT0ZH1=GS1.1.1744644977.2.1.1744646522.60.0.0; zoalang=vn; zcv4=eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxNDE2MDkiLCJkYXRhIjoia0pwTVpxX1hxSTJYY2VVYjlrcWI3YUVpTncyZFNpd3VoOEM4bDY3OXFuTSIsImlhdCI6MTc0NDY0Nzg4NCwiZXhwIjoxNzQ0NjQ4NDg0fQ.pGu96wexlbGixjvbHnUowEcJCRBQi-DKqMp3_Beb_5k; _ga_907M127EPP=GS1.1.1744644009.5.1.1744647897.59.0.0; _ga=GA1.2.1254485722.1743519547; _gat=1; developer_xsrf_token=eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJfY3NyZiIsImRhdGEiOiIxNzQ0NjQ3OTA2IiwiaWF0IjoxNzQ0NjQ3OTA2LCJleHAiOjE3NDQ2NTE1MDZ9.S8BVNm-7StJ_SUrsJDITzbPCJhZ1evWtlo32t2Yn-cA^" ^
//  -H ^ "origin: https://developers.zalo.me^" ^
//  -H ^ "priority: u=1, i^" ^
//  -H ^ "referer: https://developers.zalo.me/tools/token-debugger^" ^
//  -H ^ "sec-ch-ua: ^\^"Google Chrome^\^";v=^\^"135^\^", ^\^"Not-A.Brand^\^";v=^\^"8^\^", ^\^"Chromium^\^";v=^\^"135^\^"^" ^
//  -H ^"sec-ch-ua-mobile: ?0^" ^
//  -H ^"sec-ch-ua-platform: ^\^"Windows^\^"^" ^
//  -H ^"sec-fetch-dest: empty^" ^
//  -H ^"sec-fetch-mode: cors^" ^
//  -H ^"sec-fetch-site: same-origin^" ^
//  -H ^"user-agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/135.0.0.0 Safari/537.36^" ^
//  --data-raw ^"access_token=LLzM5SmyEtjJMr4Io3azVrPGLnNBBsrt2aXNNV5iR3KUIbeOvIXH9nDAI2ttBtu02XbD5TKzO7TF3cT2aYLDLbGCVbo_0cz9GpGEKRm62LHI3IDDdIq8KcL5E4NXVXDIVdypNQztM71nGr9ygrvhKtCyNM2AF6Tb51TCSjK4U4K-7LLPo7WqHoTwE4tgU3TLUt45KBzX94X-UXP8dMKa97rk5Z-pLdOGJ35bCAiXHn1q8diCh4KEC51TFIAIM08NLre37CnkOpupPcyJh0vkS6flQMYYMtfqP6vwI9SDKoTzQd8-f65FCcLSN1kfMc4mINyBDRaT1pX053mmdofJ1K0AHHIW17a9MG5qPQS87bavBtLAt3OVIG0JAsN002SA600w2SOa61vU0WDsfZyNOqmi16-lI6HYgL5OMiWWEdy^"
