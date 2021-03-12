<template>
  <div>
    <button @click="harmony.unshift(getEmptyEntry())" type="button">Add to start</button>
    <div v-for="entry in harmony" class="flex">
      <textarea type="text" v-model="entry.value"></textarea>
      <div>
        <p v-html="process(entry.value, true)"></p>
        <p v-html="process(entry.value)"></p>

      </div>
    </div>
    <button @click="harmony.push(getEmptyEntry())" type="button">Add to end</button>
    <pre contenteditable @blur="harmony = JSON.parse($event.target.innerText)" v-html="localStoreAndShow(harmony)"></pre>
  </div>
</template>

<script>
import kjvjson from 'assets/kjv.json'
import { ref } from '@nuxtjs/composition-api'

// console.log(kjvjson[39][0][0]);

export default {
  setup () {
    const harmony = ref(JSON.parse(localStorage.getItem('harmony')) ?? [])
    const bookId = {
      mt: 40,
      mk: 41,
      lu: 42,
      jo: 43,
      matthew: 40,
      mark: 41,
      luke: 42,
      john: 43
    }

        const punctuation = '!?.,;:';

    function localStoreAndShow (harmony) {
      let data = JSON.stringify(harmony);

      localStorage.setItem('harmony', data);

      return data;
    }


    function getEmptyEntry () {
      return {
        value: '',
        flags: [],
      }
    }

    function removeCharacters(string, characters) {
      characters.split('').forEach(function (character) {
        string = string.replaceAll(character, '');
      });

      return string;
    }

    function process (string, flat = false) {
      const passages = string.split('\n');
      let texts = ''

      passages.forEach(function (passage, index) {
        const deCapitalize = passage.includes('_');
        const capitalize = passage.includes('^');
        let addEndingPunctuation = passage.includes('+');
        if (addEndingPunctuation) {
          addEndingPunctuation = passage.split('+')[1];
        }
        const setEndingPunctuation = addEndingPunctuation ? false : getPunctuationChange(passage);
        if (setEndingPunctuation) {
          passage = passage.slice(0, -1);
        }
        passage = removeCharacters(passage, '_^');
        passage = removeCharacters(passage, '_^');
        const reference = passage.trim().toLowerCase().split(':')
        const chapter = reference[0].split(' ') ?? null
        const book = chapter[0] ?? null
        const ch = chapter[1] ?? null
        const v = reference[1] ?? null
        const w = reference[2] ?? null

        // console.log({
        //   passage,
        //   reference,
        //   chapter,
        //   book,
        //   ch,
        //   v,
        //   w
        // })

        if (reference && chapter && book && ch && v) {
          let text = ''

          if (v.includes('-')) {
            const split = v.split('-')
            const start = split[0] ?? null
            const end = split[1] ?? null

            if (start && end) {
              for (let i = start; i <= end; i++) {
                const verse = kjvjson[bookId[book] - 1][ch - 1][i - 1] + ' '
                if (verse !== 'undefined ') {
                  text += verse
                } else {
                  text += '<span class="error">Verse ' + i + ' not found!</span> '
                }
              }
            }
          } else {
            text = kjvjson[bookId[book] - 1][ch - 1][v - 1]
            if (w) {
              if (w.includes('-')) {
                const split = w.split('-')
                const start = split[0] ?? null
                const end = split[1] ?? null
                let words = ''

                if (start && end) {
                  for (let i = Number(start); i <= Number(end); i++) {
                    const word = text.split(' ')[i - 1] + ' '

                    if (word !== 'undefined ') {
                      words += word
                    } else {
                      words += '<span class="error">Word ' + i + ' of ' + passage + ' not found!</span> '
                    }
                  }
                  text = words
                }
              } else {
                text = text.split(' ')[w - 1] + ' '
              }
            } else {
              text += ' '
            }
          }
          text = removeCharacters(text, '@_{}[]');
          // console.log({passage, setEndingPunctuation, addEndingPunctuation, capitalize, deCapitalize});
          if (setEndingPunctuation) {
            text = text.trim().slice(0, -1) + setEndingPunctuation + ' ';
          } else if (addEndingPunctuation) {
            text = text.trim() + addEndingPunctuation + ' ';
          }
          if (deCapitalize) {
            text = deCapitalizeFirstLetter(text);
          } else if (capitalize) {
            text = capitalizeFirstLetter(text);
          }
          if (text !== 'undefined') {
            if (flat) {
              texts += text;
            } else {
              texts += '<span><t>' + text + '</t><r>' + passage + '</r></span>';
            }
          }
        }
      })

      if (texts === '') {
        return 'nothing found'
      }

      return texts;
    }

    function getPunctuationChange(string) {
      let punct = false;

      punctuation.split('').forEach(function (p) {
        const lastChar = string.slice(-1);
        console.log({lastChar});
        if (lastChar === p) {
          console.log (lastChar + ' is ' + p);
          punct = p;
        }
      });

      return punct;
    }

    function capitalizeFirstLetter (string) {
      return string.charAt(0).toUpperCase() + string.slice(1);
    }

    function deCapitalizeFirstLetter (string) {
      console.log ('decaptailizing ' + string.charAt(0));
      return string.charAt(0).toLowerCase() + string.slice(1);
    }

    return {
      harmony,
      process,
      getEmptyEntry,
      localStoreAndShow
    }
  }
}
</script>

<style>
span {
  display: inline-block;
  margin-right: 1ex;
}
t, r {
  display: block;
}
r {
  font-size: 0.5rem;
}
textarea {
  all: revert;
  width: 30%;
}
.error {
  color: red;
  zoom: 0.75;
}
button {
    all: revert;
    width: 100%;
    zoom: 1.5;
}
body {
    padding: 1rem;
}
input {
    all: revert;
    margin: 1rem;
}
p {
    all: revert;
}
</style>
