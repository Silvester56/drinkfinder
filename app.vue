<template>
  <div>
    <Header />
    <Question v-if="remainingCocktails.length !== numberOfcocktailsAtTheEnd" :phrase="remainingQuestions[0].phrase" :button-list="remainingQuestions[0].buttonList" @answer="handleAnswer"/>
    <div v-else class="cocktail-preview-list">
      <div v-for="c, i in remainingCocktails" :class="`cocktail-preview ${getColorClass(c)}`">
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
    phrase: "Favorite spirit?",
    buttonList: buttonListFromTags([Tags.VODKA, Tags.GIN, Tags.TEQUILA])
  },
  {
    phrase: "Choose a spirit",
    buttonList: buttonListFromTags([Tags.COGNAC, Tags.CACHAÇA, Tags.RUM])
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
    phrase: "Favorite country?",
    buttonList: buttonListFromTags([Tags.FRANCE, Tags.MEXICO, Tags.RUSSIA, Tags.UK])
  },
  {
    phrase: "What Hogwarts house would you go to?",
    buttonList: [{text: "Gryffindor", tag: Tags.RED}, {text: "Hufflepuff", tag: Tags.YELLOW}, {text: "Ravenclaw", tag: Tags.BLUE}, {text: "Slytherin", tag: Tags.GREEN}]
  },
  {
    phrase: "Lime or lemon?",
    buttonList: buttonListFromTags([Tags.LIME, Tags.LEMON])
  }
].sort(() => 0.5 - Math.random());

const tagsOnCondition = (condition, tagsIfConfition, tagsIfNotCondition) => {
  if (condition) {
    return tagsIfConfition;
  }
  return tagsIfNotCondition ? tagsIfNotCondition : [];
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
    cocktail.tags = cocktail.tags.concat(tagsOnCondition(tmpString.includes(tag.toString()), [tag]));
  });
  cocktail.tags = cocktail.tags.concat(tagsOnCondition(tmpString.search(/\bGIN\b/) !== -1, [Tags.GIN]));
  cocktail.tags = cocktail.tags.concat(tagsOnCondition(tmpString.search(/\bCOLA\b/) !== -1, [Tags.COLA, Tags.BLACK]));
  cocktail.tags = cocktail.tags.concat(tagsOnCondition(tmpString.includes("LIME"), [Tags.LIME, Tags.GREEN]));
  cocktail.tags = cocktail.tags.concat(tagsOnCondition(tmpString.includes("LEMON"), [Tags.LEMON, Tags.YELLOW]));
  cocktail.tags = cocktail.tags.concat(tagsOnCondition(tmpString.includes("RUM"), [Tags.RUM]));
  cocktail.tags = cocktail.tags.concat(tagsOnCondition(tmpString.includes("TEQUILA"), [Tags.TEQUILA, Tags.MEXICO]));
  cocktail.tags = cocktail.tags.concat(tagsOnCondition(tmpString.includes("VODKA"), [Tags.VODKA]));
  cocktail.tags = cocktail.tags.concat(tagsOnCondition(tmpString.includes("COGNAC"), [Tags.COGNAC]));
  cocktail.tags = cocktail.tags.concat(tagsOnCondition(tmpString.includes("FRENCH"), [Tags.FRANCE]));
  cocktail.tags = cocktail.tags.concat(tagsOnCondition(tmpString.includes("MOSCOW"), [Tags.RUSSIA]));
  cocktail.tags = cocktail.tags.concat(tagsOnCondition(tmpString.includes("LONDON"), [Tags.UK]));
  cocktail.tags = cocktail.tags.concat(tagsOnCondition(cocktail.ingredients.length <= 3, [Tags.FEW_INGREDIENTS], [Tags.MANY_INGREDIENTS]));
  cocktail.tags = cocktail.tags.concat(tagsOnCondition(tmpString.includes("SALT"), [Tags.SALT]));
  cocktail.tags = cocktail.tags.concat(tagsOnCondition(tmpString.includes("SUGAR"), [Tags.SUGAR]));
  cocktail.tags = cocktail.tags.concat(tagsOnCondition(tmpString.includes("COFFEE") || tmpString.includes("ESPRESSO"), [Tags.COFFEE, Tags.BLACK]));
  cocktail.tags = cocktail.tags.concat(tagsOnCondition(tmpString.includes("VANILLA"), [Tags.VANILLA]));
  cocktail.tags = cocktail.tags.concat(tagsOnCondition(tmpString.search(/\bRED\b/) !== -1 || tmpString.includes("BLOODY"), [Tags.RED]));
  cocktail.tags = cocktail.tags.concat(tagsOnCondition(tmpString.includes("GREEN"), [Tags.GREEN]));
  cocktail.tags = cocktail.tags.concat(tagsOnCondition(tmpString.includes("WHITE"), [Tags.WHITE]));
  return cocktail;
}).sort(() => 0.5 - Math.random());

const cocktailScore = (cocktail, tagList) => {
  return cocktail.tags.reduce((acc, cur) => acc + (tagList.includes(cur) ? 1 : 0), 0);
};

const questionScore = (question, tagList) => {
  return question.buttonList.reduce((acc, cur) => acc + (tagList.includes(cur.tag) ? 1 : 0), 0);
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
      numberOfcocktailsToConsiderForNextQuestion: 5
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
      this.remainingQuestions.sort((a, b) => questionScore(b, tagsToConsider) - questionScore(a, tagsToConsider));
      if (this.questionsLeft === 0) {
        this.remainingCocktails.splice(this.numberOfcocktailsAtTheEnd);
        [this.remainingCocktails[0], this.remainingCocktails[1]] = [this.remainingCocktails[1], this.remainingCocktails[0]];
      }
    },
    printRemainingCocktails() {
      this.remainingCocktails.slice(0, this.numberOfcocktailsToConsiderForNextQuestion).forEach(c => console.log(c.name, c.tags.join(", "), "\n"));
    },
    getColorClass(cocktail) {
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
  --tomato: #f55d3e;
  --lavender-web: #e5e5f7;
  --emerald: #00dc82;
  --jonquil: #f7cb15;
  --licorice: #1e120b;
  --air-force-blue: #537d8d;
  --inverted-tomato: #0aa2c1;
  --inverted-lavender-web: #1a1a08;
  --inverted-emerald: #ff237d;
  --inverted-jonquil: #0834ea;
  --inverted-air-force-blue: #ac8272;
  --inverted-licorice: #e1edf4;
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
}

.cocktail-preview-list .cocktail-preview:hover {
  color: #ffffff;
}

.cocktail-preview-list .cocktail-preview.tomato { background-color: var(--tomato) }
.cocktail-preview-list .cocktail-preview.lavender-web { background-color: var(--lavender-web) }
.cocktail-preview-list .cocktail-preview.emerald { background-color: var(--emerald) }
.cocktail-preview-list .cocktail-preview.jonquil { background-color: var(--jonquil) }
.cocktail-preview-list .cocktail-preview.air-force-blue { background-color: var(--air-force-blue) }
.cocktail-preview-list .cocktail-preview.licorice { background-color: var(--licorice); color: #ffffff; }
.cocktail-preview-list .cocktail-preview.tomato:hover { background-color: var(--inverted-tomato) }
.cocktail-preview-list .cocktail-preview.lavender-web:hover { background-color: var(--inverted-lavender-web) }
.cocktail-preview-list .cocktail-preview.emerald:hover { background-color: var(--inverted-emerald) }
.cocktail-preview-list .cocktail-preview.jonquil:hover { background-color: var(--inverted-jonquil) }
.cocktail-preview-list .cocktail-preview.air-force-blue:hover { background-color: var(--inverted-air-force-blue) }
.cocktail-preview-list .cocktail-preview.licorice:hover { background-color: var(--inverted-licorice); color: #000000 }

</style>