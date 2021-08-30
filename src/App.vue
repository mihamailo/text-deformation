<template>
  <div class="container">
    <Filter :filter="state.filter" :filterChange="filterChange" />
    <div class="result card">
      <div class="card-body" :class="state.textAlign">
        <div v-if="typeof state.text === 'object'">
          <div v-for="(item, index) in state.text" v-bind:key="index">
            {{ item }}
          </div>
        </div>
        <div v-else>{{ state.text }}</div>
      </div>
    </div>
    <Input :setText="setText" />
  </div>
</template>

<script>
import { reactive, onMounted, onUpdated } from "vue";
import Input from "./components/Input";
import Filter from "./components/Filter";
import html2canvas from "html2canvas";
import { jsPDF } from "jspdf";

export default {
  name: "App",
  components: {
    Input,
    Filter,
  },
  setup() {
    const state = reactive({
      text: "Deformation result",
      filter: "Зазеркалить",
      textAlign: "text-center",
    });
    function array_compare(a, b) {
      // if lengths are different, arrays aren't equal
      if (a.length != b.length) return false;

      for (let i = 0; i < a.length; i++) if (a[i] != b[i]) return false;

      return true;
    }

    const shuffelText = (value) => {
      function shuffelWord(word) {
        var array = word.split("");
        var oldArray = array.slice();

        var currentIndex = array.length,
          temporaryValue,
          randomIndex;
        var lastLetter = array[currentIndex - 1];
        function randomize(count) {
          while (count !== currentIndex) {
            if (
              lastLetter === "!" ||
              lastLetter === "," ||
              lastLetter === "?" ||
              lastLetter === ":" ||
              lastLetter === "."
            ) {
              currentIndex -= 1;
            }
            randomIndex = Math.floor(Math.random() * currentIndex);
            currentIndex -= 1;

            temporaryValue = array[currentIndex];
            array[currentIndex] = array[randomIndex];
            array[randomIndex] = temporaryValue;

            return array;
          }
        }

        if (array.length <= 3) {
          randomize(1);
        } else {
          randomize(array.length - 1);
        }

        if (array_compare(array, oldArray)) {
          array[1] = oldArray[0];
          array[0] = oldArray[1];
        }

        return array.join("");
      }

      const arrStrings = value.split(" ");
      let newArray = [];
      arrStrings.forEach((item) => {
        const suffelItem = shuffelWord(item);
        newArray.push(suffelItem);
      });

      const newText = newArray.join(" ");
      return newText;
    };

    const mirrorText = (value) => {
      // var res = backString(value.toLowerCase());

      function backString(aString) {
        var last = aString.length - 1;
        var res = new Array(aString.length);
        for (var i = last; i >= 0; --i) {
          var c = aString.charAt(i);
          res[last - i] = c;
        }
        return res.join("");
      }

      function reverseStrings(text) {
        let array = text.split(" ");
        let countSym = 0;
        let newArray = [];
        let prevId = 0;
        let textLength = text.replace(/\s/g, "").length;
        let singleString = "";
        let stringItem;
        array.map((item, id) => {
          stringItem = array.slice(prevId, id + 1).join(" ");
          prevId = id + 1;
          console.log(item);
          if (countSym > 120) {
            singleString += ` ${stringItem}`;
            newArray.push(backString(singleString));
            singleString = "";
            countSym = 0;
          } else {
            singleString += ` ${stringItem}`;
            countSym += item.length;
          }
        });
        console.log(newArray);
        // here we load string that have less than 50 symbols
        newArray.push(backString(singleString));

        // console.log(newContent);
        return newArray;
      }

      const text = reverseStrings(value);
      return text;
    };

    const onShuffelText = (value) => {
      state.text = shuffelText(value);
      state.textAlign = "text-start";
    };

    const onMirrorText = (value) => {
      state.text = mirrorText(value);
      state.textAlign = "text-end";
    };

    const onDefault = (value) => {
      state.text = value;
      state.textAlign = "text-center";
    };

    onUpdated(() => {
      html2canvas(document.querySelector(".result .card-body")).then(function (
        canvas
      ) {
        document.body.appendChild(canvas);
        canvas.style.width = "100vw";
        canvas.style.width = "2500px";
        canvas.style.visibility = "hidden";
        canvas.style.position = "absolute";
        canvas.style.left = "-1000%";
        var imgData = canvas.toDataURL("image/jpeg", 1.0);
        var pdf = new jsPDF({
          orientation: "l",
          unit: "px",
          format: [canvas.width, canvas.height * 2],
        });

        pdf.addImage(imgData, "jpeg", 0, 0, canvas.width, canvas.height);
        pdf.save("download.pdf");
      });
    });

    const setText = (value) => {
      if (value === null || value === "") {
        onDefault("Deformation result");
        return;
      }
      switch (state.filter) {
        case "Зазеркалить":
          return onMirrorText(value);
        case "Перемешать":
          return onShuffelText(value);
        default:
          return onDefault;
      }
    };

    const filterChange = (type) => {
      state.filter = type;
      const inputValue = document.querySelector("textarea").value;
      if (!inputValue) {
        return;
      }
      setText(inputValue);
    };

    return {
      state,
      setText,
      filterChange,
    };
  },
};
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  padding-top: 20px;
}
.result {
  min-height: 60vh;
  font-size: 18px;
}
</style>
