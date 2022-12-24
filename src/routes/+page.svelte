<script>
    import { Configuration, OpenAIApi } from 'openai';
    import { jsPDF } from "jspdf";
    import "./Myfont-Regular 2-normal.js";
    import "./num1-normal.js";
    import "./num2-normal.js";
    import "./txt2-normal"
    import "./txt3-normal"
    const configuration = new Configuration({
   apiKey: "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",
   });
   const openai = new OpenAIApi(configuration);
   Array.prototype.random = function () {
  return this[Math.floor((Math.random()*this.length))];
}
let input = "";
let summary = "";
let words = "DONT CHANGE THIS VALUE UNLESS YOU WANT A SPECIFIC AMOUNT OF WORDS PER POINT";
async function summarize() {
   //gpt3 call
   let wrd = words ? (words != "DONT CHANGE THIS VALUE UNLESS YOU WANT A SPECIFIC AMOUNT OF WORDS PER POINT") : 10;
   var pars = input.split("\n\n").map(word => word.trim())
   if (pars.length > 4) {
    let sum = ``;
        for (let i = 0; i < pars.length; i++) {
            let gptInp = `
         Summarize the following text by writing one short concise Bullet Point:
         Write MAXIMUM ${wrd} words per bullet point. 
         WRITE LITTLE WORDS BE CONCISE

            Text: ${pars[i]}
            Bullet Point:
         `;
            const response = await openai.createCompletion({
                model: "text-curie-001",
                prompt: gptInp,
                temperature: 0.7,
                max_tokens: 80,
                top_p: 1,
                frequency_penalty: 0,
                presence_penalty: 0,
                stop: ["\n"]
                });
                console.log(response.data.choices[0].text);
            sum += await response.data.choices[0].text + "\n";
        }
        console.log(sum);
        summary = sum.split("\n").filter(point => point != "").map(word => word.trim().replaceAll("-", "").trim()).filter(x => x.trim().length != 0);
        return;
    }

   var gptInp = `
   Summarize the following text by writing one bullet point per paragraph .
   ONLY ${pars.length} bullet points are allowed. (No more)
    USE SIMPLE WORDS AND SHORT SENTENCES.

   Text: ${input}
   
   Bullet points:
   `;
   var gptOut = "";
   const response = await openai.createCompletion({
    model: "text-davinci-002",
    prompt: gptInp,
    temperature: 0.7,
    max_tokens: 100,
    top_p: 1,
    frequency_penalty: 0,
    presence_penalty: 0,
    });
    console.log(response)
    console.log(await response.data.choices[0].text);
    summary = await response.data.choices[0].text.split("\n").filter(point => point != "").map(word => word.trim().replaceAll("-", "").trim());
}
async function downloadpdf() {
    let x = 20;
    let y = 20;
    var doc = new jsPDF();
    doc.setFontSize(15);
    doc.setFont("Myfont-Regular 2","txt2")
    let fonts = ["num1"];
    let numfonts = ["num1"]
    // doc.text(20, 30, "-"+summary.join("\n -"), {maxWidth: 180});
    
    for (let i = 0; i < summary.length; i++) {
        y += [8,9,10,11,12].random();
        x = 20;
        doc.text(x, y, "-");
        x = 24;
        for (let j = 0; j < summary[i].length; j++) {
            if (["0","1","2","3","4","5","6","7","8","9"].includes(summary[i][j])) {
                doc.setFont(numfonts.random(), "normal");
            }
            else {
                let fnt = fonts.random();
                doc.setFont(fnt, "normal");
            }
            if (x + 3 > 180 ) {
                y += 5;
                x = 20;
            }
            if ( y > 280) {
                doc.addPage();
                y = 20;
                x = 20;
            }
            x += 2.4;
            if (summary[i][j] == " ") {
            x += .4;
            }
            else {
            doc.text(x, y, summary[i][j]);
            }
        }
    }
    doc.save('summary.pdf');
}
</script>

  <div>
    <section class="section">
        <div class="container">
            <h1 class="title">Note Summarizer</h1>
            <label class="label">Words per point:</label>
                    <input class="input" type="text" bind:value={words} />
            <div class="field">
                <label class="label">Enter your notes:</label>
                <div class="control">
                    
                    <textarea style="height:250px" class="textarea" bind:value={input}></textarea>
                </div>
            </div>
            <div class="field">
                <div class="control">
                    <button class="button is-primary" on:click={summarize}>Summarize</button>
                </div>
                <div class="control">
                </div>
            </div>
            {#if summary.length > 0}
            <div class="field">
                <label class="label">Summary:</label>
                {#each summary as item}
                <li>{item}</li>
                {/each}
                <button class="button is-primary is-blue" on:click={downloadpdf}>Download HANDWRITING PDF</button>
            </div>
            {/if}
        </div>
    </section>
</div>

