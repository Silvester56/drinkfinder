<template>
  <div>
    <Header />
    <Question v-if="remainingCocktails.length !== numberOfcocktailsAtTheEnd" :phrase="remainingQuestions[0].phrase" :button-list="remainingQuestions[0].buttonList" @answer="handleAnswer"/>
    <div v-else class="cocktail-preview-list">
      <div v-for="c, i in remainingCocktails" :class="`cocktail-preview background-${backgroundClass[i]}`">
        <h2>{{ c.name }}</h2>
      </div>
    </div>
    <Footer />
  </div>
</template>

<script>

import Footer from "./components/Footer.vue";
import Question from "./components/Question.vue";
import Header from "./components/Header.vue";
import cocktails from "./mock/cocktails.json";

const enumValue = (name) => Object.freeze({toString: () => name});
const Tags = Object.freeze({
  VODKA: enumValue("VODKA"),
  GIN: enumValue("GIN"),
  RUM: enumValue("RUM"),
  TEQUILA: enumValue("TEQUILA"),
  COGNAC: enumValue("COGNAC"),
  WHISKEY: enumValue("WHISKEY"),
  WHISKY: enumValue("WHISKY"),
  CACHAÇA: enumValue("CACHAÇA"),
  CHAMPAGNE: enumValue("CHAMPAGNE"),
  PROSECCO: enumValue("PROSECCO"),
  TRIPLE_SEC: enumValue("TRIPLE_SEC"),
  COINTREAU: enumValue("COINTREAU"),
  FEW_INGREDIENTS: enumValue("FEW_INGREDIENTS"),
  MANY_INGREDIENTS: enumValue("MANY_INGREDIENTS"),
  MEXICO: enumValue("MEXICO"),
  FRANCE: enumValue("FRANCE"),
  RUSSIA: enumValue("RUSSIA"),
  UK: enumValue("UK"),
  SALT: enumValue("SALT"),
  SUGAR: enumValue("SUGAR"),
  COFFEE: enumValue("COFFEE"),
  CHOCOLATE: enumValue("CHOCOLATE"),
  VANILLA: enumValue("VANILLA"),
  LIME: enumValue("LIME"),
  LEMON: enumValue("LEMON"),
  COLA: enumValue("COLA"),
  BLACK: enumValue("BLACK"),
  WHITE: enumValue("WHITE"),
  RED: enumValue("RED"),
  GREEN: enumValue("GREEN"),
  BLUE: enumValue("BLUE"),
  YELLOW: enumValue("YELLOW"),
});

const buttonListFromTags = (tags) => tags.map(t => {
  return {text: t.toString().toLowerCase(), tag: t}
});

let allQuestions = [
  {
    phrase: "Salt or sugar?",
    buttonList: buttonListFromTags([Tags.SALT, Tags.SUGAR])
  },
  {
    phrase: "Chocolate or vanilla?",
    buttonList: buttonListFromTags([Tags.CHOCOLATE, Tags.VANILLA])
  },
  {
    phrase: "Do you need an energy boost?",
    buttonList: [{text: "Yes", tag: Tags.COFFEE}, {text: "No"}]
  },
  {
    phrase: "Are you lazy?",
    buttonList: [{text: "Yes", tag: Tags.FEW_INGREDIENTS}, {text: "No", tag: Tags.MANY_INGREDIENTS}]
  },
  {
    dynamic: true,
    phrase: "Choose a spirit",
    buttonList: buttonListFromTags([Tags.VODKA, Tags.GIN, Tags.TEQUILA, Tags.COGNAC, Tags.CACHAÇA, Tags.RUM])
  },
  {
    phrase: "Champagne or Prosecco ?",
    buttonList: buttonListFromTags([Tags.CHAMPAGNE, Tags.PROSECCO])
  },
  {
    phrase: "Whisky or Whiskey ?",
    buttonList: buttonListFromTags([Tags.WHISKY, Tags.WHISKEY])
  },
  {
    dynamic: true,
    phrase: "Favorite country?",
    buttonList: buttonListFromTags([Tags.FRANCE, Tags.MEXICO, Tags.RUSSIA, Tags.UK])
  },
  {
    dynamic: true,
    phrase: "What Hogwarts house would you go to?",
    buttonList: [{text: "Gryffindor", tag: Tags.RED}, {text: "Hufflepuff", tag: Tags.YELLOW}, {text: "Ravenclaw", tag: Tags.BLUE}, {text: "Slytherin", tag: Tags.GREEN}]
  },
  {
    phrase: "Lime or lemon?",
    buttonList: buttonListFromTags([Tags.LIME, Tags.LEMON])
  }
].sort((a, b) => a.dynamic === b.dynamic ? 0.5 - Math.random() : (b.dynamic ? -1 : 1));

const concatOnCondition = (arr, condition, tagsIfConfition, tagsIfNotCondition) => {
  let elementsToAdd = condition ? tagsIfConfition : tagsIfNotCondition;
  if (elementsToAdd) {
    return arr.concat(elementsToAdd.filter(e => !arr.includes(e)));
  }
  return arr;
};

let allCocktails = cocktails.map((cocktail, index) => {
  let tmpString = cocktail.ingredients.reduce((acc, cur) => `${cur.ingredient}\n${acc}`, `${cocktail.name}\n${cocktail.method}\n`);
  if (cocktail.garnish) {
    tmpString = `${tmpString}\n${cocktail.garnish}`;
  }
  tmpString = tmpString.toUpperCase();
  cocktail.id = index;
  cocktail.tags = [];
  [Tags.WHISKEY, Tags.WHISKY, Tags.CACHAÇA, Tags.CHAMPAGNE, Tags.PROSECCO, Tags.TRIPLE_SEC, Tags.COINTREAU].forEach(tag => {
    cocktail.tags = concatOnCondition(cocktail.tags, tmpString.includes(tag.toString()), [tag]);
  });
  cocktail.tags = concatOnCondition(cocktail.tags, tmpString.search(/\bGIN\b/) !== -1, [Tags.GIN]);
  cocktail.tags = concatOnCondition(cocktail.tags, tmpString.search(/\bCOLA\b/) !== -1, [Tags.COLA, Tags.BLACK]);
  cocktail.tags = concatOnCondition(cocktail.tags, tmpString.includes("LIME"), [Tags.LIME, Tags.GREEN]);
  cocktail.tags = concatOnCondition(cocktail.tags, tmpString.includes("LEMON"), [Tags.LEMON, Tags.YELLOW]);
  cocktail.tags = concatOnCondition(cocktail.tags, tmpString.includes("RUM"), [Tags.RUM]);
  cocktail.tags = concatOnCondition(cocktail.tags, tmpString.includes("TEQUILA"), [Tags.TEQUILA, Tags.MEXICO]);
  cocktail.tags = concatOnCondition(cocktail.tags, tmpString.includes("VODKA"), [Tags.VODKA]);
  cocktail.tags = concatOnCondition(cocktail.tags, tmpString.includes("COGNAC"), [Tags.COGNAC]);
  cocktail.tags = concatOnCondition(cocktail.tags, tmpString.includes("FRENCH"), [Tags.FRANCE]);
  cocktail.tags = concatOnCondition(cocktail.tags, tmpString.includes("MOSCOW"), [Tags.RUSSIA]);
  cocktail.tags = concatOnCondition(cocktail.tags, tmpString.includes("LONDON"), [Tags.UK]);
  cocktail.tags = concatOnCondition(cocktail.tags, cocktail.ingredients.length <= 3, [Tags.FEW_INGREDIENTS], [Tags.MANY_INGREDIENTS]);
  cocktail.tags = concatOnCondition(cocktail.tags, tmpString.includes("SALT"), [Tags.SALT]);
  cocktail.tags = concatOnCondition(cocktail.tags, tmpString.includes("SUGAR"), [Tags.SUGAR]);
  cocktail.tags = concatOnCondition(cocktail.tags, tmpString.includes("COFFEE") || tmpString.includes("ESPRESSO"), [Tags.COFFEE, Tags.BLACK]);
  cocktail.tags = concatOnCondition(cocktail.tags, tmpString.includes("VANILLA"), [Tags.VANILLA]);
  cocktail.tags = concatOnCondition(cocktail.tags, tmpString.search(/\bRED\b/) !== -1 || tmpString.includes("BLOODY"), [Tags.RED]);
  cocktail.tags = concatOnCondition(cocktail.tags, tmpString.includes("GREEN"), [Tags.GREEN]);
  cocktail.tags = concatOnCondition(cocktail.tags, tmpString.includes("WHITE"), [Tags.WHITE]);
  return cocktail;
}).sort(() => 0.5 - Math.random());

const cocktailScore = (cocktail, tagList) => {
  return cocktail.tags.reduce((acc, cur) => acc + (tagList.includes(cur) ? 1 : 0), 0);
};

const questionScore = (question, tagList) => {
  return (question.buttonList.length >= 2 && question.buttonList.every(cur => tagList.includes(cur.tag))) ? Math.random() : -1;
};

export default {
  name: "default",
  components: {
    Header,
    Question,
    Footer
  },
  data () {
    return {
      remainingQuestions: allQuestions,
      remainingCocktails: allCocktails,
      currentTagList: [],
      questionsLeft: 0,
      numberOfcocktailsAtTheEnd: 3,
      numberOfcocktailsToConsiderForNextQuestion: 5,
      backgroundClass: Array.from(Array(8)).map((_, i) => i + 1).sort(() => 0.5 - Math.random())
    }
  },
  methods: {
    handleAnswer(tag) {
      let tagsToConsider;

      this.questionsLeft--;
      this.remainingQuestions.splice(0, 1);
      if (!tag) {
        return;
      }
      this.currentTagList.push(tag);
      this.remainingCocktails.sort((a, b) => cocktailScore(b, this.currentTagList) - cocktailScore(a, this.currentTagList));
      tagsToConsider = this.remainingCocktails.slice(0, this.numberOfcocktailsToConsiderForNextQuestion).reduce((acc, cur) => {
        let tmpArray = [];
        cur.tags.forEach(t => {
          if (!acc.includes(t)) {
            tmpArray.push(t);
          }
        });
        return acc.concat(tmpArray);
      }, []);
      this.printRemainingCocktails();
      console.log("\n");
      console.log(tagsToConsider.join(", "));
      console.log("\n");
      this.remainingQuestions = this.remainingQuestions.map(q => {
        if (q.dynamic) {
          q.buttonList = q.buttonList.filter(b => tagsToConsider.includes(b.tag));
        }
        return q;
      });
      this.remainingQuestions.sort((a, b) => questionScore(b, tagsToConsider) - questionScore(a, tagsToConsider));
      if (this.questionsLeft === 0) {
        this.remainingCocktails.splice(this.numberOfcocktailsAtTheEnd);
        [this.remainingCocktails[0], this.remainingCocktails[1]] = [this.remainingCocktails[1], this.remainingCocktails[0]];
      }
    },
    printRemainingCocktails() {
      this.remainingCocktails.slice(0, this.numberOfcocktailsToConsiderForNextQuestion).forEach(c => console.log(c.name, c.tags.join(", "), "\n"));
    },
    getClass(cocktail) {
      if (cocktail.tags.includes(Tags.BLACK)) {
        return "licorice";
      }
      if (cocktail.tags.includes(Tags.RED)) {
        return "tomato";
      }
      if (cocktail.tags.includes(Tags.GREEN)) {
        return "emerald";
      }
      if (cocktail.tags.includes(Tags.BLUE)) {
        return "air-force-blue";
      }
      if (cocktail.tags.includes(Tags.YELLOW)) {
        return "jonquil";
      }
      return "lavender-web";
    }
  },
  created() {
    this.questionsLeft = 3 + Math.floor(2 * Math.random());
  }
}

</script>

<style>


body {
  margin: 0;
  padding: 0;
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
}

.cocktail-preview-list {
  background-color: #cccccc;
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 1rem;
  height: calc(100vh - 200px);
}

.cocktail-preview-list .cocktail-preview {
  width: 300px;
  height: 300px;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  background-color: #000000;
  background-size: 100%;
  background-position: center;
  background-repeat: no-repeat;
  -webkit-filter: grayscale(100%);
  filter: grayscale(100%);
  transition: all .5s;
}

.cocktail-preview-list .cocktail-preview:hover {
  color: #ffffff;
  background-size: 110%;
}

.cocktail-preview-list .cocktail-preview.background-1 {background-image: url("./assets/1.jpg")}
.cocktail-preview-list .cocktail-preview.background-2 {background-image: url("./assets/2.jpg")}
.cocktail-preview-list .cocktail-preview.background-3 {background-image: url("./assets/3.jpg")}
.cocktail-preview-list .cocktail-preview.background-4 {background-image: url("./assets/4.jpg")}
.cocktail-preview-list .cocktail-preview.background-5 {background-image: url("./assets/5.jpg")}
.cocktail-preview-list .cocktail-preview.background-6 {background-image: url("./assets/6.jpg")}
.cocktail-preview-list .cocktail-preview.background-7 {background-image: url("./assets/7.jpg")}
.cocktail-preview-list .cocktail-preview.background-8 {background-image: url("./assets/8.jpg")}

</style>