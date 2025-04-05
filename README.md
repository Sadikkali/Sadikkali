const express = require("express");
const axios = require("axios");
require("dotenv").config();

const app = express();
app.use(express.json());

const WHATSAPP_TOKEN = process.env.WHATSAPP_TOKEN;
const VERIFY_TOKEN = process.env.VERIFY_TOKEN;

// Webhook Verification
app.get("/webhook", (req, res) => {
  const mode = req.query["hub.mode"];
  const token = req.query["hub.verify_token"];
  const challenge = req.query["hub.challenge"];

  if (mode && token === VERIFY_TOKEN) {
    res.status(200).send(challenge);
  } else {
    res.sendStatus(403);
  }
});

// Receive WhatsApp Messages
app.post("/webhook", async (req, res) => {
  const body = req.body;

  if (body.object) {
    const message = body.entry[0].changes[0].value.messages[0];
    if (message) {
      const from = message.from;
      const text = message.text ? message.text.body.toLowerCase() : "";
      let reply = "Hello! How can I assist you?";

      if (text.includes("schedule")) {
        reply = "Your work schedule is updated!";
      } else if (text.includes("delivery")) {
        reply = "Please provide the postal code for the delivery location.";
      }

      try {
        await axios.post(
          `https://graph.facebook.com/v17.0/${process.env.PHONE_NUMBER_ID}/messages`,
          {
            messaging_product: "whatsapp",
            to: from,
            text: { body: reply },
          },
          { headers: { Authorization: `Bearer ${WHATSAPP_TOKEN}` } }
        );
      } catch (error) {
        console.error(
          "Error sending message:",
          error.response ? error.response.data : error.message
        );
      }
    }
    res.sendStatus(200);
  } else {
    res.sendStatus(404);
  }
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`WhatsApp bot is running on port ${PORT}`);
});
