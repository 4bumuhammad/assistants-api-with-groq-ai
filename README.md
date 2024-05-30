## Create a super fast AI assistant with Groq (Without a database)

&nbsp;

Reference : <br />
<!-- - Docs | Create a super fast AI assistant with Groq (Without a database)
  <pre>https://serpapi.com/blog/create-super-fast-ai-assistant-with-groq/</pre> -->

- Github | nodejs code "Create an AI assistant with Groq AI without database."
  <pre>https://github.com/hilmanski/assistants-api-with-groq-ai</pre>

&nbsp;

## &#x1F530; Installing Multiple Versions of Node.js Using nvm

<pre>
    ❯ nvm install 20.13.1

        Downloading and installing node v20.13.1...
        Downloading https://nodejs.org/dist/v20.13.1/node-v20.13.1-darwin-arm64.tar.xz...
        ############################################################################################################################################################################################################### 100.0%
        Computing checksum with sha256sum
        Checksums matched!
        Now using node v20.13.1 (npm v10.5.2)
    
    # or
    ❯ nvm use v20.13.1
      Now using node v20.13.1 (npm v10.5.2)

    ❯ nvm list

            v14.17.3
            v16.14.2
            v18.17.0
        ->     v20.13.1


    ❯ node --version
        v20.13.1
</pre>

&nbsp; 

## Install dependencies
```bash
  ❯ npm i cors express groq-sdk dotenv --save
```

&nbsp;

## Add API Key
Create a new .env file. Add your Groq API key in this file like this:
```bash
GROQ_API_KEY=YOUR_GROQ_API_KEY
```

&nbsp;

&nbsp;

## Code

<pre>
  ❯ vim index.js


      const express = require('express');
      const cors = require('cors')

      const app = express();
      app.use(express.json());
      app.use(cors()) 
      const port = 3000

      require("dotenv").config();
      const { GROQ_API_KEY } = process.env;

      // GROQ Setup
      const Groq = require("groq-sdk");
      const groq = new Groq({
          apiKey: GROQ_API_KEY
      });

      async function chatWithGroq(userMessage, latestReply, messageHistory) {
          let messages = [{
              role: "user",
              content: userMessage
          }]

          if(messageHistory != '') {
              messages.unshift({
                  role: "system",
                  content: `Our conversation's summary so far: """${messageHistory}""". 
                          And this is the latest reply from you """${latestReply}"""`
              })
          }

          console.log('original message', messages)

          const chatCompletion = await groq.chat.completions.create({
              messages,
              model: "llama3-8b-8192"
          });

          const respond = chatCompletion.choices[0]?.message?.content || ""
          return respond
      }

      async function summarizeConversation(message, reply, messageSummary) {
          let content = `Summarize this conversation 
                          user: """${message}""",
                          you(AI): """${reply}"""
                        `
          // For N+1 message
          if(messageSummary != '') {
              content = `Summarize this conversation: """${messageSummary}"""
                          and last conversation: 
                          user: """${message}""",
                          you(AI): """${reply}"""
          }

          const chatCompletion = await groq.chat.completions.create({
              messages: [
                  {
                      role: "user",
                      content: content
                  }
              ],
              model: "llama3-8b-8192"
          });

          const summary = chatCompletion.choices[0]?.message?.content || ""
          console.log('summary: ', summary)
          return summary
      }

      app.post('/chat', async (req, res) => {
          const { message, latestReply, messageSummary } = req.body;

          console.log('message from client: ', message)
          const reply = await chatWithGroq(message, latestReply, messageSummary)
          const summary = await summarizeConversation(message, reply, messageSummary)
          res.send({
              reply,
              summary
          })
      })

      app.listen(port, () => {
        console.log(`Example app listening on port ${port}`)
      })
</pre>


&nbsp;

## Run

<pre>
    ❯ node index.js
        Example app listening on port 3000
</pre>

&nbsp;

Open Terminal and commands
<pre>
  ❯ curl -X POST http://localhost:3000/chat \
      -H "Content-Type: application/json" \
      -d '{
          "message": "Hello, how are you?",
          "latestReply": "I am fine, thank you!",
          "messageSummary": "We were discussing about your well-being."
          }'

  {"reply":"I am fine, thank you!","summary":"There is no conversation to summarize. The text you provided is a snippet of a conversation, but it appears to be the start of a conversation rather than a complete conversation."}
</pre>

&nbsp;

<pre>
  ❯ curl -X POST http://localhost:3000/chat \
          -H "Content-Type: application/json" \
          -d '{
              "message": "Berikan bahan-bahan untuk memasak nasi goreng yang enak dengan menggunakan bahasa indonesia"
            }'

  {"reply":"Berikut adalah bahan-bahan yang diperlukan untuk memasak nasi goreng yang enak:\n\nBahan:\n\n* 2 cup nasi putih yang sudah disimpan sebelumnya\n* 1 telur ayam\n* 1 buah wortel, cacah\n* 1 siung bawang bombay, cacah\n* 1 siung bawang merah, cacah\n* 2 tangkai seledri, cacah\n* 1 sdt garam\n* 1/2 sdt lada bubuk\n* 1 sdt kecap ikan\n* 1 sdt minyak wijen\n* 1 sdm margarine\n* 1 sdm minyak goreng\n* Gula pasir secukupnya\n* Pepaya yang telah disimpan sebelumnya (bisa dihilangkan jika tidak suka)\n\nInstruksi:\n\n1. Panaskan minyak goreng dalam wajan dengan api sedang.\n2. Tumis bawang bombay dan bawang merah hingga layu dan harum.\n3. Tambahkan wortel, seledri, garam, lada, dan kecap ikan. Aduk rata.\n4. Tambahkan nasi putih, telur ayam, dan gula pasir. Aduk hingga rata.\n5. Tambahkan margarine dan minyak wijen. Aduk hingga rata.\n6. Masak hingga nasi goreng tersebut jadi, serta korek dengan gurih.\n7. Hidangkan panas-panas atau dingin dengan pepaya yang telah disimpan sebelumnya.\n\nDemikian, nasi goreng yang enak dan nikmat sudah siap. Bon appetit!","summary":"This conversation is an exchange between a user (using a Indonesian phrase) and a language model (AI) to provide ingredients and recipe for a delicious fried rice dish (Nasi Goreng). The AI responds with a list of ingredients and a step-by-step cooking instruction in Indonesian. The ingredients include cooked white rice, egg, carrot, onions, garlic, scallions, salt, black pepper, fish sauce, sesame oil, margarine, vegetable oil, and sugar. The cooking steps involve pan-frying the onions and garlic, then adding the carrots, scallions, salt, and black pepper. The cooked rice, egg, and sugar are then added and mixed well. Finally, margarine and sesame oil are added, and the dish is cooked until flavorful and served hot or cold, optionally topped with pickled papaya."}
</pre>

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

---

&nbsp;

<div align="center">
    <img src="./gambar-petunjuk/well_done.png" alt="well_done" style="display: block; margin: 0 auto;">
</div> 

&nbsp;

---

&nbsp;

&nbsp;

&nbsp;