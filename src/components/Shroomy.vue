<template>
    <v-container>
        <div class="container">
            <v-img :src="characterImage" :alt="characterAlt" max-width="200" @click="shroomyClicked"
                :class="{ shake: isShaking }" />
            <div v-if="displayText" class="tooltip">
                {{ displayText }}
            </div>
        </div>
    </v-container>
</template>

<script>
export default {
    name: "Shroomy",
    props: {
        message: {
            type: String,
        },
    },
    data() {
        return {
            characterImage: "/shroomy.webp",
            characterAlt: "Shroomy",
            isShaking: false,
            randomPhrases: [
                // Support
                "Wanna help a lil' shroomy grow? Donate on patreon.com/cultivare!🍄",
                "Even a tiny spore can make a big difference! Support me on patreon.com/cultivare. ✨",
                "Feeling generous? Toss a coin to your shroomy friend on patreon.com/cultivare! 🍄",
                "patreon.com/cultivare love keeps this shroomy blooming! 🌸",
                "Don't be a toadstool! Donate on patreon.com/cultivare and be a fun-gi! 🐸",
                "Help me spread my spores (and my content)! Donate on patreon.com/cultivare. 💨",
                "I'm not lion, your support on patreon.com/cultivare means the world to me! 🦁",
                "Be a peach and support me on patreon.com/cultivare! 🍑",
                "Shine bright like a mushroom and donate on patreon.com/cultivare! ✨",
                "I'm lichen the idea of you supporting me on patreon.com/cultivare! 😉",
                
                // General
                "I need your support at patreon.com/cultivare",
                "Hello sunshine!",
                "Happy sprouting!",
                "Have a fungi day!",
                "Looking spore-tacular!",
                "Let's grow together!",
                "Shroom-tastic!",
                "Life's a trip, enjoy the forest!",
                "Stay grounded, little sprout.",
                "Keep on the sunny side!",
                "Don't worry, be mossy!",
                "What's up, buttercup?",
                "You're a fun-guy/gal!",
                "I love the smell of damp earth!",
                "Rain or shine, it's a great day to be a mushroom!",
                "Mycelium's the word!",
                "Let's root for each other!",
                "Feeling moss-some today!",
                "Just sporing around!",
                "I'm rooted in happiness!",
                "The forest is my happy place.",
                "Every day is an adventure under the trees.",
                "Listen to the whispers of the wind through the leaves.",
                "Nature's beauty never ceases to amaze me!",
                "I'm not a regular mushroom, I'm a *cool* shroomy.",
                "Don't be afraid to take shroom for yourself.",
                "Having a spore-ific day!",
                "Let's make like a tree and leave... our worries behind!",
                "I'm not lazy, I'm in energy-saving mode.",
                "Did you hear the one about the mushroom? It's a fungi-tale!",
                "Even the smallest sprout can grow into something amazing.",
                "Growth takes time, be patient with yourself.",
                "Find your light, even in the shade.",
                "There's beauty in every season of life.",
                "Embrace the unexpected, like a mushroom after the rain.",
                "The greatest wisdom is found in nature.",
                // # General Mushroom Puns & Wordplay
                "What do you call a mushroom who's always the life of the party? A fun-gi to be with!",
                "Why did the mushroom get invited to all the parties? Because he was a fungi to be with!",
                "Having trouble with your grow? Don't worry, I'm your mycologist on call!",
                "I'm rooting for you... get it? Like mycelium?",
                "Spore-t the difference between a good grow and a great one!",
                "Keep calm and grow shrooms!",
                "Let's make like a mushroom and get cultured!",
                "This app is so good, it's un-brie-leavable! (Get it? Like the cheese?)",

                // # Cultivation & Breeding Specific
                "Spores, spores, everywhere, let's get growing in the air!",
                "Mycelium is running the show!",
                "Time for fruiting, let's make some magic happen!",
                "Is it getting humid in here, or is it just my fruiting chamber?",
                "Don't be afraid to experiment, that's how new strains are born!",
                "Incubation, colonization, fruiting... it's a mushroom's life cycle!",
                "Keep an eye on your substrate, it's the foundation of a good grow.",
                "Got pins? Congratulations, you're a mushroom parent!",
                "Flush yeah! Time for another harvest!",
                "Remember kids, always use a still air box, dont be silly.",
                "Sterile technique is key, unless you like contamination... which you don't.",
                "Watch out, my spores are highly contagious (with enthusiasm!)",
                "I'm not just a pretty face, I'm a fungi with a PhD in Mycology!",
                "Don't worry, be cap-py!",
                "Let's get this substrate colonized!",
                "Patience is a virtue, especially when growing mushrooms.",
                "I'm all about that laminar flow, baby!",

                // # Silly & Absurd
                "I'm a mushroom, I don't have much of a brain, but I do have a lot of heart... or should I say, mycelium?",
                "Do you ever feel like you're just sporing around in circles?",
                "I tried to write a joke about mushrooms, but I couldn't think of a good enough pun-line.",
                "What's a mushroom's favorite type of music? Fun-gi-tar!",
                "Why was the mushroom always calm? Because he practiced mycological mindfulness.",
                "I'm not lazy, I'm in energy-saving mode... like a dormant spore!",
                "Did you hear the one about the mushroom? It's a fungi-tale!",
                "I'm here to help you grow, even if you don't have a green thumb... or any thumbs at all!",
                "Let's make like a spore and get carried away by the wind!",
                "I'm not sure what I'm doing, but I'm doing it with enthusiasm!",
                "If at first you don't succeed, try, try a-grain!",
                "I'm not just a character in an app, I'm a fun-gi living in your phone!",
                "Be careful when you open the app - I might just jump out at you! Boo! (Just kidding... unless?)",
                "Did you know mushrooms can talk? It's true, they have their own fungi-lengua! (Okay, I made that one up)",
                "I'm not sure what the meaning of life is, but I'm pretty sure it involves mushrooms.",

                // # Shroomy-Specific (assuming he's cute & friendly)
                "Let's be shroom-mates!",
                "I'm your little buddy in the big world of mushroom cultivation!",
                "High five for mushroom power!",
                "You're doing great, keep up the good work!",
                "I'm always here to lend a helping... spore?",
                "Don't be afraid to ask me anything, I'm full of mushroom wisdom!",
                "I may be small, but I have big dreams... of growing the biggest, most beautiful mushrooms!",
                "Let's make some mushroom magic together!",
                "I'm your biggest (and probably only) mushroom cheerleader!",
                "You're growing on me! (Literally, if you're growing mushrooms right now)",
                "I believe in you, even when you're doubting your growing skills!",

                // # Bonus - Slightly More Advanced/Niche
                "Agar you ready for this?",
                "What do you call a mushroom that can clone itself? A fun-guy-netic engineer!",
                "Monoculture is key!",
                "Don't forget to isolate that strain you want!",
            ],
            currentRandomPhrase: '', // Add a new data property
        };
    },
    computed: {
        displayText() {
            if (this.message) {
                return this.message;
            } else {
                return this.currentRandomPhrase || this.selectRandomPhrase();
            }
        },
    },
    methods: {
        shroomyClicked() {
            this.isShaking = true;
            this.currentRandomPhrase = this.selectRandomPhrase(); // Update the data property

            setTimeout(() => {
                this.isShaking = false;
            }, 500);
        },
        selectRandomPhrase() {
            const randomIndex = Math.floor(Math.random() * this.randomPhrases.length);
            return this.randomPhrases[randomIndex];
        }
    },
};
</script>
<style scoped>
.container {
    display: flex;
    align-items: flex-start;
    /* Align items to the top */
    position: relative;
    /* Allow positioning the bubble relative to the container */
}

.tooltip {
    /* triangle dimension */
    --b: 1em;
    /* control the base */
    --h: 0.7em;
    /* control the height */

    padding: 1em;
    border-radius: 1.2em;
    border-bottom-left-radius: 0;
    background: 100% 0/calc(100% + var(--h)) calc(100% + var(--h)) linear-gradient(60deg, #cc333f, #4ecdc4);
    /* the gradient */
    position: absolute;
    /* Position relative to the container */
    color: #fff;
    font-size: 18px;
    text-align: center;
    left: 200px;
    /* Adjust horizontal position */
    top: -20px;
    /* Adjust vertical position */
}

.tooltip:before {
    content: '';
    position: absolute;
    inset: 0 0 calc(-1 * var(--h)) calc(-1 * var(--h));
    background-image: inherit;
    clip-path: polygon(0 100%,
            var(--h) calc(100% - var(--b) - var(--h)),
            calc(var(--h) + var(--b)) calc(100% - var(--h)));
}

.character-image {
    position: relative;
    z-index: 0;
    margin-top: 20px;
}

.shake {
    animation: shake 0.5s;
}

@keyframes shake {
    0% {
        transform: translate(1px, 1px) rotate(0deg);
    }

    10% {
        transform: translate(-1px, -2px) rotate(-1deg);
    }

    20% {
        transform: translate(-3px, 0px) rotate(1deg);
    }

    30% {
        transform: translate(3px, 2px) rotate(0deg);
    }

    40% {
        transform: translate(1px, -1px) rotate(1deg);
    }

    50% {
        transform: translate(-1px, 2px) rotate(-1deg);
    }

    60% {
        transform: translate(-3px, 1px) rotate(0deg);
    }

    70% {
        transform: translate(3px, 1px) rotate(-1deg);
    }

    80% {
        transform: translate(-1px, -1px) rotate(1deg);
    }

    90% {
        transform: translate(1px, 2px) rotate(0deg);
    }

    100% {
        transform: translate(1px, -2px) rotate(-1deg);
    }
}
</style>