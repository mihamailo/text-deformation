<template>
  <div class="container">
    <div class="mb-3 d-flex justify-content-between">
      <Filter
        :filter="state.filter"
        :filterChange="filterChange"
        :buttons="state.buttons"
      />
      <button @click="downloadPdf" class="btn btn-success mt-2">Скачать</button>
    </div>
    <div class="result card">
      <div class="result__body card-body" :class="state.textAlign">
        <div v-if="typeof state.text === 'object'">
          <div
            v-for="(item, index) in state.text"
            v-bind:key="index"
            :class="state.hiddenSide"
          >
            {{ item }}
          </div>
        </div>

        <div v-else>{{ state.text }}</div>

        <div
          id="randomResult"
          v-bind:class="
            state.filter === 'Случайные буквы' ? 'd-block' : 'd-none'
          "
        ></div>
      </div>
    </div>
    <Input :setText="setText" />
  </div>
</template>

<script>
import { reactive } from "vue";
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
      text: "Результат деформации",
      filter: "Безпробелов",
      textAlign: "text-center",
      buttons: [
        "Перемешать",
        "Отзеркалить",
        "Случайные буквы",
        "Безпробелов",
        "Перекрыто сверху",
        "Перекрыто снизу",
      ],
      hiddenSide: "",
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
          if (res[i] === ")") res[i] = "(";
          else if (res[i] === "(") res[i] = ")";
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
        let singleString = "";
        let stringItem;
        array.map((item, id) => {
          stringItem = array.slice(prevId, id + 1).join(" ");
          prevId = id + 1;

          let spaceCount = stringItem.length + 1;
          console.log(spaceCount);
          if (countSym > 75 - spaceCount) {
            // Можно отнять количество элементов строки( = количеству пробелов)
            singleString += ` ${stringItem}`;
            newArray.push(backString(singleString));
            singleString = "";
            countSym = 0;
          } else {
            singleString += ` ${stringItem}`;
            countSym += item.length;
          }
        });
        // here we load string that have less than 50 symbols
        newArray.push(backString(singleString));

        return newArray;
      }

      const text = reverseStrings(value);
      return text;
    };

    const randomLetter = (value) => {
      const arrStrings = value.split(" ");

      function str_rand() {
        var result = "";
        var words = "йцукенгшщзхъфывапролджэячсмитьбю";
        var max_position = words.length - 1;
        for (let i = 0; i < 1; ++i) {
          let position = Math.floor(Math.random() * max_position);
          result = result + words.substring(position, position + 1);
        }
        return result;
      }

      const newArray = arrStrings.map((word, id, arr) => {
        const letters = word.split("");

        let letterArray = [];
        for (let i = 0; i < letters.length; i++) {
          letterArray.push(letters[i]);
          if (letters[i + 1] != undefined) {
            letterArray.push(`<span>${str_rand()}</span>`);
          }
        }
        return letterArray.join("");
      });

      const newText = newArray.join(" ");

      return newText;
    };

    const spaceless = (value) => {
      let newText = value.replace(/\s/g, "");

      return newText;
    };

    const getStrings = (value) => {
      let array = value.split(" ");
      let countSym = 0;
      let newArray = [];
      let prevId = 0;
      let singleString = "";
      let stringItem;
      array.map((item, id) => {
        stringItem = array.slice(prevId, id + 1).join(" ");
        prevId = id + 1;

        let spaceCount = stringItem.length + 1;
        if (countSym > 75 - spaceCount) {
          singleString += ` ${stringItem}`;
          newArray.push(singleString);
          singleString = "";
          countSym = 0;
        } else {
          singleString += ` ${stringItem}`;
          countSym += item.length;
        }
      });
      newArray.push(singleString);
      return newArray;
    };

    const onShuffelText = (value) => {
      state.text = shuffelText(value);
      state.textAlign = "text-start";
    };

    const onMirrorText = (value) => {
      state.text = mirrorText(value);
      state.textAlign = "text-end";
    };

    const onRandomLetter = (value) => {
      state.textAlign = "text-start";
      state.text = randomLetter("");
      document.querySelector("#randomResult").innerHTML = randomLetter(value);
    };

    const onSpaceless = (value) => {
      state.textAlign = "text-start";
      state.text = spaceless(value);
    };

    const onTopHide = (value) => {
      state.textAlign = "text-start";
      state.text = getStrings(value);
    };

    const onBottomHide = (value) => {
      state.textAlign = "text-start";
      state.text = getStrings(value);
    };

    const onDefault = (value) => {
      state.text = value;
      state.textAlign = "text-center";
    };

    const downloadPdf = () => {
      html2canvas(document.querySelector(".result__body")).then(function (
        canvas
      ) {
        document.body.appendChild(canvas);
        canvas.style.width = "3000px";
        canvas.style.height = "3000px";
        canvas.style.visibility = "hidden";
        canvas.style.position = "absolute";
        canvas.style.left = "-1000%";
        var imgData = canvas.toDataURL("image/jpeg", 1.0);
        var pdf = new jsPDF({
          orientation: "l",
          unit: "px",
          format: [canvas.width, canvas.width],
        });

        pdf.addImage(imgData, "jpeg", 0, 0, canvas.width, canvas.height);
        pdf.save("download.pdf");
      });
    };

    const setText = (value) => {
      if (value === null || value === "") {
        onDefault("Результат деформации");
        return;
      }
      switch (state.filter) {
        case "Отзеркалить":
          return onMirrorText(value);
        case "Перемешать":
          return onShuffelText(value);
        case "Случайные буквы":
          return onRandomLetter(value);
        case "Безпробелов":
          return onSpaceless(value);
        case "Перекрыто сверху":
          return onTopHide(value);
        case "Перекрыто снизу":
          return onBottomHide(value);
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
      downloadPdf,
    };
  },
  beforeUpdate() {
    const { state } = this;
    console.log(state.hiddenSide);
    switch (state.filter) {
      case "Перекрыто сверху":
        return (state.hiddenSide = "top-hidden");
      case "Перекрыто снизу":
        return (state.hiddenSide = "bottom-hidden");
      default:
        return (state.hiddenSide = "");
    }
  },
};
</script>

<style lang="scss">
#app {
  font-family: "Jost", sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #000;
  padding-top: 20px;
}
.result {
  min-height: 60vh;
  font-size: 24px;
  background-size: 150px;
  position: relative;
  &__img {
    position: absolute;
    left: 10px;
    top: 10px;
    width: 180px;
  }
}

#randomResult span {
  opacity: 0.6;
}

.container {
  max-width: 1300px;
}

.top-hidden {
  position: relative;
  &:after {
    content: "";
    background: #fff;
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 44%;
  }
}
.bottom-hidden {
  position: relative;
  &:after {
    content: "";
    background: #fff;
    position: absolute;
    bottom: 0;
    left: 0;
    width: 100%;
    height: 44%;
  }
}
</style>
