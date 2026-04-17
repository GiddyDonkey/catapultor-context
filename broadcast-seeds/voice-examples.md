# Fried Ocean Voice Examples

This file contains gold/silver/bronze tier examples of Fried Ocean content across categories, used as few-shot examples during generation.

**Status:** Template — Jason is curating this over time. Broadcast should seed `broadcast_voice_examples` with whatever is present here at build time, and Jason will add more examples as they're curated.

**Tier meanings:**
- **Gold:** Best-in-class examples that defined the voice; use as primary few-shot examples
- **Silver:** Solid examples worth referencing but not flagship
- **Bronze:** Acceptable examples for volume when gold/silver pool is thin

---

## Headlines

### Gold tier

- Kissing booth underperforms at herpes conference
- After exodus of members, gym asks man with very large penis to change at home
- Men’s obsession with oversized testicle implants outpaces women’s interest in large testicles
- Epidemic of stoned squirrels plaguing California suburbs
- Man’s Etch A Sketch masterpiece ruined after ground shipping snafu
- Bayonet manufacturers giddy as calls for new US Civil War grow louder
- Expert: When your dog starts talking, it’s probably time to stop drinking
- In baffling display of ignorance, 64% adults believe ‘Star Wars’ historically accurate events
- Elves frustrated that Santa jolly AF throughout layoff announcement
- Dancing pig nothing more than publicity stunt, says talking horse
- Experts: Fat shaming babies great way to teach at early age how stuff works around here
- Cleaning up after your dog, neighborly or liberal hoax?
- Aliens Unsure What to Do With 8 Billion Anal Probes as Humans Face Extinction
- Men’s Obsession With Giant Testicle Implants Vastly Outpaces Women’s Desire to See Them
- Over 80 Million Americans Under BO Smell Advisory
- Study: Teenage Boys’ Interest in Large Breasts Shows No Signs of Decline
- Dog Envious Watching Owner Repeatedly Pet Himself
- Woman in Mid-Birth Sets Confusing Start to Beauty Pageant
- With Democrats Gone, Texas GOP Turns to Brisket to Meet Quorum Requirements
- Trump Fires Secretary of the Interior After Couch Fails to Fit Through Door
- Man Fighting Fire With Fire Gravely Misinformed
- Woman Without Gag Reflex Finds It Surprisingly Easy To Make Friends At Summer Camp
- Overweight Dog Deeply Concerned by Owner’s Salivating Stare
- Brothel’s BOGO Promotion Exceeding Expectations
- Amish Themed Porn Site Off To Sluggish Start
- Man Politely Waits Until After Taking Sacrament To Adjust His Testicles
- Man Smuggling Candy in Anus Highly Overestimates Movie Theater Security
- Errant Erection Causes Man to Lose Babysitting Gig
- Man Awakes From Wet Dream to Find Most of the Other Airplane Passengers Staring at Him
- Study: 72% of Adults Convinced Shakespeare Invented Electricity
- Time Traveler’s Dream of Riding Dinosaur Ends in Bloody Pulp
- Study Reveals 72% Believe Dinosaurs Went Extinct After Losing Revolutionary War
- Cheerful Throw Pillow Really Livens Up Local Meth Den
- Couple Willing To Include Gimp In Home Sale For The Right PriceTrump Taps President Camacho to Deliver Next State of the Union
- BREAKING: Trump Airline Avoids Iranian Airspace 34 Years Before War, Claims Exceptional Foresight
- Poll Finds 103% Of Respondents Believe Voter Fraud Occurring
- Woman Reports Mild Arousal After Reading Neighbor’s Crude Wi-Fi Name
- Tardigrade Yawns As Humanity Continues Slow March Toward Extinction
- New Fertility Technique Allows Parents to Preselect Child’s Political Alignment
- Airlines Introduce Optional Chloroform Service for Uninterrupted In-Flight Sleep Experience
- Man Unsure Why Cartoon Duck Keeps Popping In Head While Masturbating
- Child Learns the Hard Way Cactus Not Fucking Around
- Lizard Doing Push-Ups Still Weak as Shit

### Silver tier

*(Jason to populate)*

### Bronze tier

*(Jason to populate)*

---

## Article intros

### Gold tier

*(Jason to populate with first 2-3 paragraphs of best past articles, with Source URL)*

### Silver tier

*(Jason to populate)*

---

## Facebook captions

### Gold tier

*(Jason to populate)*

### Silver tier

*(Jason to populate)*

---

## Instagram feed captions

### Gold tier

*(Jason to populate)*

### Silver tier

*(Jason to populate)*

---

## X posts

### Gold tier

*(Jason to populate)*

### Silver tier

*(Jason to populate)*

---

## Threads posts

### Gold tier

*(Jason to populate)*

---

## Bluesky posts

### Gold tier

*(Jason to populate)*

---

## Pinterest pin descriptions

### Gold tier

*(Jason to populate)*

---

## TikTok captions

### Gold tier

*(Jason to populate)*

---

## YouTube Shorts captions

### Gold tier

*(Jason to populate)*

---

## Newsletter blurbs

### Gold tier

*(Jason to populate)*

---

## Atoms — Punchlines

### Gold tier

- What a Fucking Hero
- Credits All the Women She Crushed Along the Way
- Maybe They Knew Something
- You Nasty for Eating That
- …and you ate it anyway
- …he won't notice

### Silver tier

*(Jason to populate)*

---

## Atoms — Quotes

### Gold tier

*(Jason to populate with memorable quotes from past articles)*

---

## Atoms — Stats / Absurd numeric precision

### Gold tier

- 34 years early
- Zero regret

### Silver tier

*(Jason to populate)*

---

## Instructions for Broadcast build session

When seeding `broadcast_voice_examples`:

1. Parse this file's headings to derive `category` enum values
2. Parse the tier subheadings to derive `performance_tier` (gold / silver / bronze)
3. Each bullet or delimited example becomes a row
4. If a category/tier is empty or contains only `(Jason to populate)` placeholder, skip — do not insert empty rows
5. If an example has a `Source URL` line, populate `source_url`
6. Re-seed is safe: on re-run, upsert by (voice_profile_id, category, content) to avoid duplicates
