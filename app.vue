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
  COLD: enumValue("COLD"),
  HOT: enumValue("HOT"),
  MEXICO: enumValue("MEXICO"),
  FRANCE: enumValue("FRANCE"),
  RUSSIA: enumValue("RUSSIA"),
  USA: enumValue("USA"),
  SALT: enumValue("SALT"),
  NO_SALT: enumValue("NO_SALT"),
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
    phrase: "Are you lazy?",
    buttonList: [{text: "Yes", tag: Tags.FEW_INGREDIENTS}, {text: "No", tag: Tags.MANY_INGREDIENTS}]
  },
  {
    phrase: "Did you already eat?",
    buttonList: [{text: "Yes", tag: Tags.STRONG}, {text: "No", tag: Tags.SOFT}]
  },
  {
    phrase: "Favorite spirit?",
    buttonList: buttonListFromTags([Tags.VODKA, Tags.GIN, Tags.RUM, Tags.TEQUILA, Tags.COGNAC])
  },
  {
    phrase: "Favorite country?",
    buttonList: buttonListFromTags([Tags.FRANCE, Tags.MEXICO, Tags.RUSSIA])
  },
  {
    phrase: "Are you freezing?",
    buttonList: [{text: "Yes", tag: Tags.HOT}, {text: "No", tag: Tags.COLD}]
  }
];

let allCocktails = cocktails.map(cocktail => {
  cocktail.tags = [];
  Object.keys(Tags).forEach(tag => {
    if (cocktail.ingredients.some(i => i.ingredient.toUpperCase().includes(tag.toString().replace("_", " ")))) {
      cocktail.tags.push(Tags[tag]);
      if (Tags[tag] === Tags.TEQUILA) {
        cocktail.tags.push(Tags.MEXICO);
      } else if (Tags[tag] === Tags.VODKA) {
        cocktail.tags.push(Tags.RUSSIA);
      }
    }
  });
  if (cocktail.garnish && cocktail.garnish.toUpperCase().includes("SALT")) {
    cocktail.tags.push(Tags.SALT);
  } else {
    cocktail.tags.push(Tags.NO_SALT);
  }
  if (cocktail.ingredients.length <= 3) {
    cocktail.tags.push(Tags.FEW_INGREDIENTS);
  } else {
    cocktail.tags.push(Tags.MANY_INGREDIENTS);
  }
  if (cocktail.name.toUpperCase().includes("FRENCH")) {
    cocktail.tags.push(Tags.FRANCE);
  }
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
      this.remainingQuestions.forEach(q => q.buttonList = q.buttonList.filter(b => this.remainingCocktails.some(c => c.tags.includes(b.tag))));
      this.remainingQuestions = this.remainingQuestions.filter(q => q.buttonList.length >= 2);
      this.remainingQuestions.sort(() => 0.5 - Math.random());
    }
  },
  created() {
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