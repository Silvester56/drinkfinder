<template>
  <div>
    <Header />
    <Question v-if="remainingQuestions.length > 0" :phrase="remainingQuestions[0].phrase" :button-list="remainingQuestions[0].buttonList" @answer="handleAnswer"/>
    <div v-else class="cocktail-list">
      <Cocktail v-for="c in remainingCocktails" :recipe="c"/>
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
  NO_SALT: enumValue("NO_SALT"),
  COFFEE: enumValue("COFFEE"),
  NO_COFFEE: enumValue("NO_COFFEE"),
});

const buttonListFromTags = (tags) => tags.map(t => {
  return {text: t.toString().toLowerCase(), tag: t}
});

let allQuestions = [
  {
    phrase: "Do you like salt?",
    buttonList: [{text: "Yes", tag: Tags.SALT}, {text: "No", tag: Tags.NO_SALT}]
  },
  {
    phrase: "Do you need an energy boost?",
    buttonList: [{text: "Yes", tag: Tags.COFFEE}, {text: "No", tag: Tags.NO_COFFEE}]
  },
  {
    phrase: "Are you lazy?",
    buttonList: [{text: "Yes", tag: Tags.FEW_INGREDIENTS}, {text: "No", tag: Tags.MANY_INGREDIENTS}]
  },
  {
    phrase: "Did you already eat?",
    buttonList: [{text: "Yes", tag: Tags.STRONG}, {text: "No", tag: Tags.SOFT}]
  },
  {
    phrase: "Favorite spirit?",
    buttonList: buttonListFromTags([Tags.VODKA, Tags.GIN, Tags.TEQUILA])
  },
  {
    phrase: "Favorite spirit?",
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
  }
].sort((a, b) => b.buttonList.length - a.buttonList.length + 0.5 - Math.random());

const tagsOnCondition = (condition, tagsIfConfition, tagsIfNotCondition) => {
  if (condition) {
    return tagsIfConfition;
  }
  return tagsIfNotCondition ? tagsIfNotCondition : [];
};

let allCocktails = cocktails.map(cocktail => {
  let tmpString = cocktail.ingredients.reduce((acc, cur) => `${cur.ingredient}\n${acc}`, `${cocktail.name}\n${cocktail.method}\n`);
  console.log(tmpString);
  if (cocktail.garnish) {
    tmpString = `${tmpString}\n${cocktail.garnish}`;
  }
  tmpString = tmpString.toUpperCase();
  cocktail.tags = [];
  [Tags.WHISKEY, Tags.WHISKY, Tags.CACHAÇA, Tags.CHAMPAGNE, Tags.PROSECCO, Tags.TRIPLE_SEC, Tags.COINTREAU].forEach(tag => {
    cocktail.tags = cocktail.tags.concat(tagsOnCondition(tmpString.includes(tag.toString()), [tag]));
  });
  cocktail.tags = cocktail.tags.concat(tagsOnCondition(tmpString.search(/\bGIN\b/) !== -1, [Tags.GIN, Tags.UK]));
  cocktail.tags = cocktail.tags.concat(tagsOnCondition(tmpString.includes("RUM"), [Tags.RUM]));
  cocktail.tags = cocktail.tags.concat(tagsOnCondition(tmpString.includes("TEQUILA"), [Tags.TEQUILA, Tags.MEXICO]));
  cocktail.tags = cocktail.tags.concat(tagsOnCondition(tmpString.includes("VODKA"), [Tags.VODKA, Tags.RUSSIA]));
  cocktail.tags = cocktail.tags.concat(tagsOnCondition(tmpString.includes("COGNAC"), [Tags.COGNAC, Tags.FRANCE]));
  cocktail.tags = cocktail.tags.concat(tagsOnCondition(tmpString.includes("FRENCH"), [Tags.FRANCE]));
  cocktail.tags = cocktail.tags.concat(tagsOnCondition(cocktail.ingredients.length <= 3, [Tags.FEW_INGREDIENTS], [Tags.MANY_INGREDIENTS]));
  cocktail.tags = cocktail.tags.concat(tagsOnCondition(tmpString.includes("SALT"), [Tags.SALT], [Tags.NO_SALT]));
  cocktail.tags = cocktail.tags.concat(tagsOnCondition(tmpString.includes("COFFEE") || tmpString.includes("ESPRESSO"), [Tags.COFFEE], [Tags.NO_COFFEE]));
  return cocktail;
});

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
    }
  },
  methods: {
    handleAnswer(tag) {
      this.remainingQuestions.splice(0, 1);
      if (!tag) {
        return;
      }
      this.remainingCocktails = this.remainingCocktails.filter(cocktail => cocktail.tags.includes(tag));
      this.printRemainingCocktails();
      this.remainingQuestions.forEach(q => q.buttonList = q.buttonList.filter(b => this.remainingCocktails.some(c => c.tags.includes(b.tag))));
      this.remainingQuestions = this.remainingQuestions.filter(q => q.buttonList.length >= 2);
      this.remainingQuestions.sort((a, b) => b.buttonList.length - a.buttonList.length + 0.5 - Math.random());
    },
    printRemainingCocktails() {
      this.remainingCocktails.forEach(c => console.log(c.name, c.tags.join(", "), "\n"));
    }
  },
  created() {
    this.printRemainingCocktails();
  }
}

</script>

<style>

body {
  margin: 0;
  padding: 0;
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
}

.cocktail-list {
  padding: 0 1rem;
  display: flex;
  gap: 1rem;
}

.cocktail-list .cocktail {
  flex: 1;
}

</style>