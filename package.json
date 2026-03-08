import express from "express";
import cors from "cors";

const app = express();
app.use(cors());
app.use(express.json());

const PORT = process.env.PORT || 3000;
const RETELL_API_KEY = process.env.RETELL_API_KEY;     // SECRET key iz Retell-a
const RETELL_AGENT_ID = process.env.RETELL_AGENT_ID;   // agent_...

// sessionId -> chat_id (memorija; dovoljno za start)
const sessions = new Map();

async function createChat(sessionId) {
  const res = await fetch("https://api.retellai.com/create-chat", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${RETELL_API_KEY}`,
    },
    body: JSON.stringify({
      agent_id: RETELL_AGENT_ID,
      metadata: { source: "webwave", sessionId },
    }),
  });

  const data = await res.json();
  if (!res.ok) throw new Error("create-chat failed: " + JSON.stringify(data));
  return data.chat_id;
}

async function createCompletion(chatId, content) {
  const res = await fetch("https://api.retellai.com/create-chat-completion", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${RETELL_API_KEY}`,
    },
    body: JSON.stringify({ chat_id: chatId, content }),
  });

  const data = await res.json();
  if (!res.ok) throw new Error("create-chat-completion failed: " + JSON.stringify(data));
  return data;
}

function extractReply(messages) {
  if (!Array.isArray(messages)) return null;
  const last = [...messages].reverse().find(
    (m) => m.role === "agent" && typeof m.content === "string"
  );
  return last?.content?.trim() || null;
}

app.get("/api/health", (_req, res) => res.json({ ok: true }));

app.post("/api/chat", async (req, res) => {
  try {
    const { message, sessionId } = req.body;

    if (!RETELL_API_KEY || !RETELL_AGENT_ID) {
      return res.status(500).json({ error: "Missing RETELL_API_KEY or RETELL_AGENT_ID" });
    }
    if (!message || typeof message !== "string") {
      return res.status(400).json({ error: "message is required" });
    }

    const sid =
      typeof sessionId === "string" && sessionId.trim() ? sessionId.trim() : "default";
    let chatId = sessions.get(sid);

    if (!chatId) {
      chatId = await createChat(sid);
      sessions.set(sid, chatId);
    }

    const completion = await createCompletion(chatId, message);
    const reply = extractReply(completion.messages) || "Trenutno nemam odgovor.";

    res.json({ reply });
  } catch (e) {
    console.error(e);
    res.status(500).json({ error: "Greška pri komunikaciji s Retell agentom." });
  }
});

app.listen(PORT, () => console.log("Listening on", PORT));
