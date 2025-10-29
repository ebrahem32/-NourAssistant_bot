import express from "express";
import fetch from "node-fetch";
import bodyParser from "body-parser";
import dotenv from "dotenv";

dotenv.config();
const app = express();
app.use(bodyParser.json());

const GEMINI_KEY = process.env.GEMINI_KEY;
const WEBHOOK_SECRET = process.env.WEBHOOK_SECRET || "DeltaNourSecretKey2025";

app.post("/webhook", async (req, res) => {
  try {
    if (req.query.secret !== WEBHOOK_SECRET)
      return res.status(401).json({ error: "unauthorized" });

    const body = req.body || {};
    let source = "unknown";
    let sender = "غير معروف";
    let text = "";
    let fileUrl = "";
    let fileType = "text";

    if (body.message) {
      source = "Telegram";
      const msg = body.message;
      sender = msg.from?.username || msg.from?.first_name || "مجهول";
      text = msg.text || "";
      if (msg.document || msg.photo) fileType = "file";
    } else if (body.app && body.message) {
      source = "WhatsApp";
      sender = body.sender || body.phone || "مجهول";
      text = body.message || "";
      if (body.file_url) {
        fileType = "file";
        fileUrl = body.file_url;
      }
    }

    const prompt =
      fileType === "text"
        ? `حلل الرسالة التالية من ${source}:\n"${text}"`
        : `تم استقبال ملف جديد من ${source}. نوع الملف: ${fileType}`;

    const gRes = await fetch(
      `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash-exp:generateContent?key=${GEMINI_KEY}`,
      {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          contents: [{ role: "user", parts: [{ text: prompt }] }],
        }),
      }
    );

    const result = await gRes.json();
    const analysis =
      result?.candidates?.[0]?.content?.parts?.[0]?.text ||
      "لم يتمكن الذكاء من التحليل.";

    console.log("📩 تحليل:", analysis);
    return res.json({ reply: "تم استلام الرسالة ✅", analysis });
  } catch (e) {
    console.error(e);
    return res.status(500).json({ error: e.message });
  }
});

const PORT = process.env.PORT || 9000;
app.listen(PORT, () => console.log(`🚀 NourAssistant running on port ${PORT}`));        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          contents: [{ role: "user", parts: [{ text: prompt }] }],
        }),
      }
    );

    const result = await gRes.json();
    const analysis =
      result?.candidates?.[0]?.content?.parts?.[0]?.text ||
      "لم يتمكن الذكاء من التحليل.";

    console.log("📩 تحليل:", analysis);
    return res.json({ reply: "تم استلام الرسالة ✅", analysis });
  } catch (e) {
    console.error(e);
    return res.status(500).json({ error: e.message });
  }
});

const PORT = process.env.PORT || 9000;
app.listen(PORT, () => console.log(`🚀 NourAssistant listening on ${PORT}`));
