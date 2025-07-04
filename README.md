# Hobi-Sitesi
// Program.cs – .NET 8 Minimal API
using System.Text;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Http;
using System.Collections.Generic;

var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

/* ---------- Ana Sayfa (Basit HTML Formu) ---------- */
app.MapGet("/", async ctx =>
{
    const string html = """
    <!DOCTYPE html>
    <html lang="tr">
    <head>
        <meta charset="utf-8" />
        <title>Hobi Önerici</title>
        <style>
            body{font-family:Arial,Helvetica,sans-serif;margin:40px;}
            label{display:block;margin-top:10px;}
            input,select{padding:6px;width:220px;}
            button{margin-top:20px;padding:10px 24px;}
        </style>
    </head>
    <body>
        <h1>Sana Uygun Hobiyi Bul</h1>
        <form method="post" action="/recommend">
            <label>Yaş: <input type="number" name="age" required /></label>

            <label>Cinsiyet:
                <select name="gender">
                    <option value="Kadın">Kadın</option>
                    <option value="Erkek">Erkek</option>
                    <option value="Diğer">Diğer</option>
                </select>
            </label>

            <label>Meslek: <input type="text" name="job" /></label>

            <label>Yaşam Tarzı:
                <select name="lifestyle">
                    <option value="aktif">Aktif</option>
                    <option value="sakin">Sakin</option>
                </select>
            </label>

            <label>Yaşadığın Yer (opsiyonel): <input type="text" name="location" /></label>

            <button type="submit">Hobileri Göster</button>
        </form>
    </body>
    </html>
    """;

    ctx.Response.ContentType = "text/html; charset=utf-8";
    await ctx.Response.WriteAsync(html, Encoding.UTF8);
});

/* ---------- Öneri Sonucu ---------- */
app.MapPost("/recommend", async ctx =>
{
    var form = await ctx.Request.ReadFormAsync();
    int age          = int.TryParse(form["age"], out var a) ? a : 0;
    string gender    = form["gender"];
    string job       = form["job"];
    string lifestyle = form["lifestyle"];
    string location  = form["location"];

    var hobbies = HobbyRecommender.Recommend(age, gender, job, lifestyle, location);

    var sb = new StringBuilder("""
    <!DOCTYPE html>
    <html lang="tr">
    <head>
        <meta charset="utf-8"/>
        <title>Önerilen Hobiler</title>
        <style>
            body{font-family:Arial,Helvetica,sans-serif;margin:40px;}
            .card{border:1px solid #ddd;border-radius:10px;padding:14px;margin-bottom:12px;box-shadow:0 1px 4px rgba(0,0,0,0.08);}
            a{display:inline-block;margin-top:20px;}
        </style>
    </head>
    <body>
        <h1>Sana Uygun Hobiler</h1>
    """);

    foreach (var h in hobbies)
        sb.Append($"""<div class="card"><strong>{h}</strong><br/>{HobbyRecommender.GetDescription(h)}</div>""");

    sb.Append("""
        <a href="/">↩ Yeni bir arama</a>
    </body>
    </html>
    """);

    ctx.Response.ContentType = "text/html; charset=utf-8";
    await ctx.Response.WriteAsync(sb.ToString(), Encoding.UTF8);
});

app.Run();

/* ---------- Basit Kural Tabanlı Öneri Motoru ---------- */
static class HobbyRecommender
{
    private static readonly Dictionary<string, string> Descriptions = new()
    {
        { "Koşu",              "Düzenli kardiyo yapmayı sağlayan, düşük maliyetli bir spordur." },
        { "Bisiklet Sürme",    "Hem ulaşım hem de egzersiz için ideal açık-hava aktivitesidir." },
        { "Yürüyüş / Trekking","Doğayla iç içe vakit geçirirken kondisyonu artırır." },
        { "Yoga",              "Esneklik, denge ve zihinsel dinginlik kazandırır." },
        { "Boyama / Çizim",    "Yaratıcılığı artıran, rahatlatıcı bir sanatsal uğraştır." },
        { "Model Uçak / Drone","Teknoloji meraklıları için eğlenceli bir hobidir." },
        { "Bahçıvanlık",       "Bitki yetiştirerek doğaya yakınlaşmanı sağlar." },
        { "Fotoğrafçılık",     "Anı yakalama ve yaratıcı kompozisyon üretmeye yöneliktir." },
        { "Oyun Geliştirme",   "Kodlama becerilerini eğlenceyle birleştiren yaratıcı bir alandır." },
        { "Ebru Sanatı",       "Geleneksel su üzerine desen oluşturma tekniğidir." },
        { "Ahşap İşçiliği",    "El becerilerini geliştiren ve özgün ürünler ortaya çıkaran uğraştır." },
        { "Dans",              "Ritim duygusunu geliştiren sosyal bir aktivitedir." }
    };

    public static List<string> Recommend(
        int    age,
        string gender,
        string job,
        string lifestyle,
        string location)
    {
        var list = new List<string>();

        /* Yaş grubuna göre */
        if      (age < 13)  list.AddRange([ "Boyama / Çizim", "Model Uçak / Drone" ]);
        else if (age <= 19) list.AddRange([ "Bisiklet Sürme", "Fotoğrafçılık", "Oyun Geliştirme" ]);
        else if (age <= 35) list.AddRange([ "Koşu", "Dans", "Fotoğrafçılık" ]);
        else if (age <= 60) list.AddRange([ "Bahçıvanlık", "Yoga", "Ahşap İşçiliği" ]);
        else                list.AddRange([ "Bahçıvanlık", "Yürüyüş / Trekking", "Ebru Sanatı" ]);

        /* Yaşam tarzına göre */
        if (lifestyle == "aktif")
            list.AddRange([ "Koşu", "Bisiklet Sürme", "Yürüyüş / Trekking", "Dans" ]);
        else
            list.AddRange([ "Boyama / Çizim", "Bahçıvanlık", "Yoga" ]);

        /* Meslek ipuçları (örnek) */
        if (job.Contains("mühendis",  StringComparison.OrdinalIgnoreCase) ||
            job.Contains("developer", StringComparison.OrdinalIgnoreCase) ||
            job.Contains("yazılım",   StringComparison.OrdinalIgnoreCase))
        {
            list.AddRange([ "Oyun Geliştirme", "Model Uçak / Drone" ]);
        }

        if (job.Contains("öğretmen", StringComparison.OrdinalIgnoreCase))
        {
            list.AddRange([ "Ebru Sanatı", "Boyama / Çizim" ]);
        }

        /* Lokasyon (örnek) – sahil/deniz yakınsa */
        if (location.Contains("sahil", StringComparison.OrdinalIgnoreCase) ||
            location.Contains("deniz", StringComparison.OrdinalIgnoreCase))
        {
            list.AddRange([ "Yürüyüş / Trekking", "Fotoğrafçılık" ]);
        }

        return new HashSet<string>(list).ToList();   // tekrarları at
    }

    public static string GetDescription(string hobby) =>
        Descriptions.TryGetValue(hobby, out var desc) ? desc : "";
}
