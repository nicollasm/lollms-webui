<template>
  <div class="container bg-bg-light dark:bg-bg-dark shadow-lg overflow-y-auto scrollbar-thin scrollbar-track-bg-light-tone scrollbar-thumb-bg-light-tone-panel hover:scrollbar-thumb-primary dark:scrollbar-track-bg-dark-tone dark:scrollbar-thumb-bg-dark-tone-panel dark:hover:scrollbar-thumb-primary active:scrollbar-thumb-secondary">
    <div class="container flex flex-row m-2">
      <div class="flex-grow m-2">
        <div class="flex-grow m-2 p-2 border border-blue-300 rounded-md border-2 border-blue-300 m-2 p-4">
          <label class="mt-2">Presets</label>
          <select v-model="selectedPreset" class="m-2 border-2 rounded-md shadow-sm w-full">
            <option v-for="preset in Object.keys(presets)" :key="preset" :value="preset">
              {{ preset }}
            </option>
          </select>
          <button class="w-6 ml-2 hover:text-secondary duration-75 active:scale-90 cursor-pointer" @click="setPreset"  title="Use preset"><i data-feather="check"></i></button>
          <button class="w-6 ml-2 hover:text-secondary duration-75 active:scale-90 cursor-pointer" @click="addPreset"  title="Add this text as a preset"><i data-feather="plus"></i></button>
          <button class="w-6 ml-2 hover:text-secondary duration-75 active:scale-90 cursor-pointer" @click="removePreset"  title="Remove preset"><i data-feather="x"></i></button>
          <button class="w-6 ml-2 hover:text-secondary duration-75 active:scale-90 cursor-pointer" @click="savePreset"  title="Save presets list"><i data-feather="save"></i></button>
          <button class="w-6 ml-2 hover:text-secondary duration-75 active:scale-90 cursor-pointer" @click="reloadPresets"  title="Reload presets list"><i data-feather="refresh-ccw"></i></button>
          
        </div>
        <div class="flex-grow m-2 p-2 border border-blue-300 rounded-md border-2 border-blue-300 m-2 p-4">
          <div class="m-0">
            <button v-show="!generating" id="generate-button" @click="generate" class="w-6 ml-2 hover:text-secondary duration-75 active:scale-90 cursor-pointer"><i data-feather="pen-tool"></i></button>
            <button v-show="generating" id="stop-button" @click="stopGeneration" class="w-6 ml-2 hover:text-secondary duration-75 active:scale-90 cursor-pointer"><i data-feather="x"></i></button>
            <button
                type="button"
                @click="startSpeechRecognition"
                :class="{ 'text-red-500': isLesteningToVoice }"
                class="w-6 hover:text-secondary duration-75 active:scale-90 cursor-pointer"
            >   
            <i data-feather="mic"></i>
            </button>
            <button
                    title="speak"
                    @click.stop="speak()"
                    :class="{ 'text-red-500': isTalking }"
                    class="w-6 hover:text-secondary duration-75 active:scale-90 cursor-pointer">
              <i data-feather="volume-2"></i>
            </button>
            <button v-show="!generating" id="export-button" @click="exportText" class="w-6 ml-2 hover:text-secondary duration-75 active:scale-90 cursor-pointer"><i data-feather="upload"></i></button>
            <button v-show="!generating" id="import-button" @click="importText" class="w-6 ml-2 hover:text-secondary duration-75 active:scale-90 cursor-pointer"><i data-feather="download"></i></button>
          
            <input type="file" id="import-input" class="hidden">
          </div>          
          <textarea v-model="text" id="text_element" class="mt-4 h-64 overflow-y-scroll w-full dark:bg-bg-dark  scrollbar-thin scrollbar-track-bg-light-tone scrollbar-thumb-bg-light-tone-panel hover:scrollbar-thumb-primary dark:scrollbar-track-bg-dark-tone dark:scrollbar-thumb-bg-dark-tone-panel dark:hover:scrollbar-thumb-primary active:scrollbar-thumb-secondary" type="text"></textarea>
          <span>Cursor position {{ cursorPosition }}</span>
                    
        </div>
        <div class="flex-grow m-2 p-2 border border-blue-300 rounded-md border-2 border-blue-300 m-2 p-4">
          <MarkdownRenderer ref="mdRender" :markdown-text="text" class="dark:bg-bg-dark">
          </MarkdownRenderer>          
        </div>
      </div>
      <div id="settings" class="border border-blue-300 bg-blue-200 mt-4 w-25 mr-2 h-full mb-10 min-w-500" style="align-items: center; height: fit-content; margin: 10px; box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1); border-radius: 4px;">
        <div id="title" class="border border-blue-600 bg-blue-300 m-0 flex justify-center items-center box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1) border-radius: 4px;">
          <h3 class="text-gray-600 mb-4 text-center m-0">Settings</h3>
        </div>
          <div class="slider-container ml-2 mr-2">
            <h3 class="text-gray-600">Temperature</h3>
            <input type="range" v-model="temperature" min="0" max="5" step="0.1" class="w-full">
            <span class="slider-value text-gray-500">Current value: {{ temperature }}</span>
          </div>
          <div class="slider-container ml-2 mr-2">
            <h3 class="text-gray-600">Top K</h3>
            <input type="range" v-model="top_k" min="1" max="100" step="1" class="w-full">
            <span class="slider-value text-gray-500">Current value: {{ top_k }}</span>
          </div>
          <div class="slider-container ml-2 mr-2">
            <h3 class="text-gray-600">Top P</h3>
            <input type="range" v-model="top_p" min="0" max="1" step="0.1" class="w-full">
            <span class="slider-value text-gray-500">Current value: {{ top_p }}</span>
          </div>
          <div class="slider-container ml-2 mr-2">
            <h3 class="text-gray-600">Repeat Penalty</h3>
            <input type="range" v-model="repeat_penalty" min="0" max="5" step="0.1" class="w-full">
            <span class="slider-value text-gray-500">Current value: {{ repeat_penalty }}</span>
          </div>
          <div class="slider-container ml-2 mr-2">
            <h3 class="text-gray-600">Repeat Last N</h3>
            <input type="range" v-model="repeat_last_n" min="0" max="100" step="1" class="w-full">
            <span class="slider-value text-gray-500">Current value: {{ repeat_last_n }}</span>
          </div>
          <div class="slider-container ml-2 mr-2">
            <h3 class="text-gray-600">Number of tokens to crop the text to</h3>
            <input type="number" v-model="n_crop" class="w-full">
            <span class="slider-value text-gray-500">Current value: {{ n_crop }}</span>
          </div>          
          <div class="slider-container ml-2 mr-2">
            <h3 class="text-gray-600">Number of tokens to generate</h3>
            <input type="number" v-model="n_predicts" class="w-full">
            <span class="slider-value text-gray-500">Current value: {{ n_predicts }}</span>
          </div>
          <div class="slider-container ml-2 mr-2">
            <h3 class="text-gray-600">Seed</h3>
            <input type="number" v-model="seed" class="w-full">
            <span class="slider-value text-gray-500">Current value: {{ seed }}</span>
          </div>
      </div>
    </div>
  </div>
  <Toast ref="toast"/>
</template>

<script>
import feather from 'feather-icons'
import axios from "axios";
import socket from '@/services/websocket.js'
import Toast from '../components/Toast.vue'
import MarkdownRenderer from '../components/MarkdownRenderer.vue';



async function showInputPanel(name, default_value="", options=[]) {
    return new Promise((resolve, reject) => {
        const panel = document.createElement("div");
        panel.className = "fixed inset-0 flex items-center justify-center bg-black bg-opacity-50 z-50";
        if(options.length===0){
          panel.innerHTML = `
            <div class="bg-white p-6 rounded-md shadow-md w-80">
                <h2 class="text-lg font-semibold mb-3">${name}</h2>
                <textarea id="replacementInput" class="w-full h-32 border rounded p-2 mb-3">${default_value}</textarea>
                <div class="flex justify-end">
                    <button id="cancelButton" class="mr-2 px-4 py-2 border rounded">Cancel</button>
                    <button id="okButton" class="px-4 py-2 bg-blue-500 text-white rounded">OK</button>
                </div>
            </div>
        `;
        }
        else{
          panel.innerHTML = `
            <div class="bg-white p-6 rounded-md shadow-md w-80">
                <h2 class="text-lg font-semibold mb-3">${name}</h2>
                <select id="options_selector" class="form-control w-full h-25 border rounded p-2 mb-3">
                  ${options.map(option => `<option value="${option}">${option}</option>`)}
                </select>
                <div class="flex justify-end">
                    <button id="cancelButton" class="mr-2 px-4 py-2 border rounded">Cancel</button>
                    <button id="okButton" class="px-4 py-2 bg-blue-500 text-white rounded">OK</button>
                </div>
            </div>
        `;          
        }


        document.body.appendChild(panel);

        const cancelButton = panel.querySelector("#cancelButton");
        const okButton = panel.querySelector("#okButton");

        cancelButton.addEventListener("click", () => {
            document.body.removeChild(panel);
            resolve(null);
        });

        okButton.addEventListener("click", () => {
            if(options.length===0){
              const input = panel.querySelector("#replacementInput");
              const value = input.value.trim();
              document.body.removeChild(panel);
              resolve(value);
            }
            else{
              const input = panel.querySelector("#options_selector");
              const value = input.value.trim();
              document.body.removeChild(panel);
              resolve(value);
            }
        });
    });
}

function replaceInText(text, callback) {
  console.log(text)
    let replacementDict = {};
    let delimiterRegex = /@<([^>]+)>@/g;
    let matches = [];
    let match;

    while ((match = delimiterRegex.exec(text)) !== null) {
      matches.push("@<"+match[1]+">@"); // The captured group is at index 1
    }
    console.log("matches")
    console.log(matches)
    matches =  [...new Set(matches)]

    async function handleReplacement(match) {
        console.log(match)
        let placeholder = match.toLowerCase().substring(2,match.length-2);
        if (placeholder !== "generation_placeholder") {
          if (placeholder.includes(":")) {
            // Special key words
            let key_words_dict={
              "all_language_options":"english:french:german:chinese:japanese:spanish:italian:russian:portuguese:swedish:danish:dutch:norwegian:slovak:czech:hungarian:polish:ukrainian:bulgarian:latvian:lithuanian:estonian:maltese:irish:galician:basque:welsh:breton:georgian:turkmen:kazakh:uzbek:tajik:afghan:sri-lankan:filipino:vietnamese:lao:cambodian:thai:burmese:kenyan:botswanan:zimbabwean:malawian:mozambican:angolan:namibian:south-african:madagascan:seychellois:mauritian:haitian:peruvian:ecuadorian:bolivian:paraguayan:chilean:argentinean:uruguayan:brazilian:colombian:venezuelan:puerto-rican:cuban:dominican:honduran:nicaraguan:salvadorean:guatemalan:el-salvadoran:belizean:panamanian:costa-rican:antiguan:barbudan:dominica's:grenada's:st-lucia's:st-vincent's:gibraltarian:faroe-islander:greenlandic:icelandic:jamaican:trinidadian:tobagonian:barbadian:anguillan:british-virgin-islander:us-virgin-islander:turkish:israeli:palestinian:lebanese:egyptian:libyan:tunisian:algerian:moroccan:bahraini:kuwaiti:saudi-arabian:yemeni:omani:irani:iraqi:afghanistan's:pakistani:indian:nepalese:sri-lankan:maldivan:burmese:thai:lao:vietnamese:kampuchean:malaysian:bruneian:indonesian:australian:new-zealanders:fijians:tongans:samoans:vanuatuans:wallisians:kiribatians:tuvaluans:solomon-islanders:marshallese:micronesians:hawaiians",
              "all_programming_language_options":"python:c:c++:java:javascript:php:ruby:go:swift:kotlin:rust:haskell:erlang:lisp:scheme:prolog:cobol:fortran:pascal:delphi:d:eiffel:h:basic:visual_basic:smalltalk:objective-c:html5:node.js:vue.js:svelte:react:angular:ember:clipper:stex:arduino:brainfuck:r:assembly:mason:lepton:seacat:bbc_microbit:raspberry_pi_gpio:raspberry_pi_spi:raspberry_pi_i2c:raspberry_pi_uart:raspberry_pi_adc:raspberry_pi_ddio"
            }
            Object.entries(key_words_dict).forEach(([key, value]) => {
              console.log(`Key: ${key}, Value: ${value}`);
              function escapeRegExp(string) {
                  return string.replace(/[.*+?^${}()|[\]\\]/g, '\\$&'); // Escape special characters
              }

              const escapedKey = escapeRegExp(key);
              const regex = new RegExp(escapedKey, 'g');
              placeholder = placeholder.replace(regex, value);
              //text = text.replace(new RegExp(key, 'g'), value);
            });

            let splitResult = placeholder.split(":");
            let name = splitResult[0];
            let defaultValue = splitResult[1] || "";
            let options = [];
            if (splitResult.length>2) {
              options = splitResult.slice(1);
            }
            let replacement = await showInputPanel(name, defaultValue, options);
            if (replacement !== null) {
                replacementDict[match] = replacement;
            }
          }
          else{
            let replacement = await showInputPanel(placeholder);
            if (replacement !== null) {
                replacementDict[match] = replacement;
            }
          }
        }else{
          //var result = confirm("generation placeholder found. Do you want to generate?\nIf you skip generation, you still can generate manually after wards");
        }
    }
    let promiseChain = Promise.resolve();

    matches.forEach(match => {
      promiseChain = promiseChain.then(() => {
        return handleReplacement(match);
      }).then(result => {
        console.log(result);
      });
    });
    promiseChain.then(() => {
      Object.entries(replacementDict).forEach(([key, value]) => {
        console.log(`Key: ${key}, Value: ${value}`);
        function escapeRegExp(string) {
            return string.replace(/[.*+?^${}()|[\]\\]/g, '\\$&'); // Escape special characters
        }

        const escapedKey = escapeRegExp(key);
        const regex = new RegExp(escapedKey, 'g');
        text = text.replace(regex, value);
        //text = text.replace(new RegExp(key, 'g'), value);
      });
      callback(text); // Call the callback after all matches are processed
    });
}

// Get input value function
function getInputValue() {
    const inputField = document.getElementById('input-field');
    return inputField ? inputField.value : '';
}

export default {
  name: 'PlayGroundView',
  data() {
    return {
      generating:false,
      isSpeaking:false,
      voices: [],    
      isLesteningToVoice:false,
      presets:{},
      selectedPreset: '',    
      cursorPosition:0,  
      text:"",
      pre_text:"",
      post_text:"",
      temperature: 0.1,
      top_k: 50,
      top_p: 0.9,
      repeat_penalty: 1.3,
      repeat_last_n: 50,
      n_crop: -1,
      n_predicts: 2000,
      seed: -1,
    };
  },
  components:{    
    Toast,
    MarkdownRenderer
  },
  mounted() {
    const text_element = document.getElementById('text_element');
    text_element.addEventListener('input', () => {
      try{
        this.cursorPosition = text_element.selectionStart;
      }
      catch{

      }
    });    
    text_element.addEventListener('click', () => {
      try{
        this.cursorPosition = text_element.selectionStart;
      }
      catch{
        
      }
    });        
    axios.get('./presets.json').then(response => {
          console.log(response.data)
          this.presets=response.data
        }).catch(ex=>{
          this.$refs.toast.showToast(`Error: ${ex}`,4,false)
        });
        // Event handler for receiving generated text chunks
        socket.on('text_chunk', data => {
            this.appendToOutput(data.chunk);
            const text_element = document.getElementById('text_element');
            text_element.scrollTo(0, text_element.scrollHeight);
        });

        // Event handler for receiving generated text chunks
        socket.on('text_generated', data => {
            // Toggle button visibility
            this.generating=false;
        });

        socket.on('generation_error', data => {
            console.log('generation_error:', data);
            this.$refs.toast.showToast(`Error: ${data}`,4,false)
            // Toggle button visibility
            this.generating=false;
        });

        

        // Event handler for successful connection
        socket.on('connect', () => {
            console.log('Connected to LoLLMs server');
            this.$store.state.isConnected=true;
            this.generating=false
        });

        // Event handler for error during text generation
        socket.on('buzzy', error => {
            console.error('Server is busy. Wait for your turn', error);
            this.$refs.toast.showToast(`Error: ${error.message}`,4,false)
            // Toggle button visibility
            this.generating=false
        });

        // Event handler for error during text generation
        socket.on('generation_canceled', error => {
            // Toggle button visibility
            this.generating=false
            console.log("Generation canceled OK")
        });

      //console.log('chatbox mnt',this.$refs)
      this.$nextTick(() => {
          feather.replace();
      });  
      
    // Speach synthesis
    // Check if speech synthesis is supported by the browser
    if ('speechSynthesis' in window) {
    this.speechSynthesis = window.speechSynthesis;

    // Load the available voices
    this.voices = this.speechSynthesis.getVoices();

    // Make sure the voices are loaded before starting speech synthesis
    if (this.voices.length === 0) {
        this.speechSynthesis.addEventListener('voiceschanged', this.onVoicesChanged);
    } else {
    }
    } else {
    console.error('Speech synthesis is not supported in this browser.');
    }


  },
  created(){

        
  },
  computed: {
    isTalking :{
        get(){
            return this.isSpeaking
        }
    },
  },
  methods:{
    onVoicesChanged() {
      // This event will be triggered when the voices are loaded
      this.voices = this.speechSynthesis.getVoices();
      },
      speak() {
          if (this.msg) {
              this.speechSynthesis.cancel();
              this.msg = null;
              this.isSpeaking = false;
              return;
          }
          let startIndex =0;
          // Set isSpeaking to true before starting synthesis
          console.log("voice on")
          this.isSpeaking = true;

          const chunkSize = 200; // You can adjust the chunk size as needed

          // Create a new SpeechSynthesisUtterance instance
          this.msg = new SpeechSynthesisUtterance();
          this.msg.pitch = this.$store.state.config.audio_pitch;

          // Optionally, set the voice and other parameters as before
          if (this.voices.length > 0) {
              this.msg.voice = this.voices.filter(voice => voice.name === this.$store.state.config.audio_out_voice)[0];
          }


          // Function to find the index of the last sentence that fits within the chunk size
          const findLastSentenceIndex = (startIndex) => {
              let txt = this.text.substring(startIndex, startIndex+chunkSize)
              // Define an array of characters that represent end of sentence markers.
              const endOfSentenceMarkers = ['.', '!', '?', '\n'];

              // Initialize a variable to store the index of the last end of sentence marker.
              let lastIndex = -1;

              // Iterate through the end of sentence markers and find the last occurrence in the txt string.
              endOfSentenceMarkers.forEach(marker => {
              const markerIndex = txt.lastIndexOf(marker);
              if (markerIndex > lastIndex) {
                  lastIndex = markerIndex;
              }
              });
              if(lastIndex==-1){lastIndex=txt.length}
              console.log(lastIndex)
              return lastIndex+startIndex+1;
          };

          // Function to speak a chunk of text
          const speakChunk = () => {
              const endIndex = findLastSentenceIndex(startIndex);
              const chunk = this.text.substring(startIndex, endIndex);
              this.msg.text = chunk;
              startIndex = endIndex + 1;
              this.msg.onend = (event) => {
                  if (startIndex < this.text.length-2) {
                      // Use setTimeout to add a brief delay before speaking the next chunk
                      setTimeout(() => {
                          speakChunk();
                      }, 1); // Adjust the delay as needed
                  } else {
                      this.isSpeaking = false;
                      console.log("voice off :",this.text.length,"  ",endIndex)
                  }
              };
              this.speechSynthesis.speak(this.msg);
          };

          // Speak the first chunk
          speakChunk();
      },    
    getCursorPosition() {
      return this.cursorPosition;
    },    
    appendToOutput(chunk){
      this.pre_text += chunk
      this.text = this.pre_text + this.post_text
    },
    generate(){
      console.log("Finding cursor position")
      this.pre_text = this.text.substring(0,this.getCursorPosition())
      this.post_text = this.text.substring(this.getCursorPosition(), this.text.length)
      var prompt = this.text.substring(0,this.getCursorPosition())
      console.log(prompt)
      // Trigger the 'generate_text' event with the prompt
      socket.emit('generate_text', { prompt: prompt, personality: -1, n_predicts: this.n_predicts , n_crop: this.n_crop,
      parameters: {
          temperature: this.temperature,
          top_k: this.top_k,
          top_p: this.top_p,
          repeat_penalty: this.repeat_penalty, // Update with desired repeat penalty value
          repeat_last_n: this.repeat_last_n, // Update with desired repeat_last_n value
          seed: parseInt(this.seed)
      }});

      // Toggle button visibility
      this.generating=true
    },
    stopGeneration(){
      // Trigger the 'cancel_generation' event
      socket.emit('cancel_text_generation',{});
    },
    exportText(){
      const textToExport = this.text;
      const element = document.createElement('a');
      const file = new Blob([textToExport], {type: 'text/plain'});
      element.href = URL.createObjectURL(file);
      element.download = 'exported_text.txt';
      document.body.appendChild(element);
      element.click();
      document.body.removeChild(element);      
    },
    importText() {
      const inputFile = document.getElementById("import-input");
      if (!inputFile) return; // If the element doesn't exist, do nothing
      inputFile.addEventListener("change", event => {
        if (event.target.files && event.target.files[0]) {
          const reader = new FileReader();
          reader.onload = () => {
            this.text = reader.result;
          };
          reader.readAsText(event.target.files[0]);
        } else {
          alert("Please select a file.");
        }
      });
      inputFile.click();
    },
    setPreset() {
      this.text = replaceInText(this.presets[this.selectedPreset], (text)=>{
        console.log("Done")
        console.log(text);
        this.text= text
      });
    },
    
    addPreset() {
      let title = prompt('Enter the title of the preset:');
      this.presets[title] =  this.text
    },
    removePreset() {
      if (this.selectedPreset) {
        delete this.presets[this.selectedPreset];
      }
    },
    savePreset() {
      axios.post("/save_presets", this.presets).then((response) => {
          console.log(response);
          this.$refs.toast.showToast(`Presets saved`,4,true)
        });
    },
    reloadPresets() {
      axios.get('./presets.json').then(response => {
          console.log(response.data)
          this.presets=response.data
        }).catch(ex=>{
          this.$refs.toast.showToast(`Error: ${ex}`,4,false)
        });
    },
    startSpeechRecognition() {
        if ('SpeechRecognition' in window || 'webkitSpeechRecognition' in window) {
            this.recognition = new (window.SpeechRecognition || window.webkitSpeechRecognition)();
            this.recognition.lang = this.$store.state.config.audio_in_language; // Set the language, adjust as needed
            this.recognition.interimResults = true; // Enable interim results to get real-time updates

            this.recognition.onstart = () => {
              this.isLesteningToVoice = true;
              this.silenceTimer = setTimeout(() => {
                  this.recognition.stop();
              }, this.silenceTimeout); // Set the silence timeout to stop recognition
            };

            this.pre_text = this.text.substring(0,this.getCursorPosition())
            this.post_text = this.text.substring(this.getCursorPosition(), this.text.length)

            this.recognition.onresult = (event) => {
              this.generated = '';

              for (let i = event.resultIndex; i < event.results.length; i++) {
                this.generated += event.results[i][0].transcript;
              }
              this.text = this.pre_text + this.generated + this.post_text; // Update the textarea with the real-time recognized words
              this.cursorPosition = this.pre_text.length + this.generated.length;
              clearTimeout(this.silenceTimer); // Clear the silence timeout on every recognized result
              this.silenceTimer = setTimeout(() => {
                  this.recognition.stop();
              }, this.silenceTimeout); // Set a new silence timeout after every recognized result
            };

            this.recognition.onerror = (event) => {
            console.error('Speech recognition error:', event.error);
            this.isLesteningToVoice = false;
            clearTimeout(this.silenceTimer); // Clear the silence timeout on error
            };

            this.recognition.onend = () => {
              console.log('Speech recognition ended.');
              this.isLesteningToVoice = false;
              this.pre_text = this.pre_text + this.generated;
              this.cursorPosition = this.pre_text.length;
              clearTimeout(this.silenceTimer); // Clear the silence timeout when recognition ends normally
            };

            this.recognition.start();
        } else {
            console.error('Speech recognition is not supported in this browser.');
        }
        },
  }
};
</script>

<style>

  select {
    width: 200px;
  }

  body {
    background-color: #fafafa;
    font-family: sans-serif;
  }

  .container {
    margin: 4px auto;
    width: 800px;
  }

  .settings {
    position: fixed;
    top: 0;
    right: 0;
    width: 250px;
    background-color: #fff;
    z-index: 1000;
    display: none;
  }

  .settings-button {
    cursor: pointer;
    padding: 10px;
    border: 1px solid #ddd;
    border-radius: 5px;
    color: #333;
    font-size: 14px;
  }

  .settings-button:hover {
    background-color: #eee;
  }

  .settings-button:active {
    background-color: #ddd;
  }

  .slider-container {
    margin-top: 20px;
  }

  .slider-value {
    display: inline-block;
    margin-left: 10px;
    color: #6b7280;
    font-size: 14px;
  }

  .small-button {
    padding: 0.5rem 0.75rem;
    font-size: 0.875rem;
  }
</style>
  