---
title: "Pokemon - Exploratory Data Analysis"
date: 2020-09-01
tags: [Python, Data Science, Pokemon]
header:
  image: "/images/pokemon_eda/Pokeball.jpg"
excerpt: "Exploring the Pokemon dataset and answer some general questions"
---

> Written by Yash Shah

# Introduction

In today's blog we will use this [dataset](https://www.kaggle.com/abcsds/pokemon) about Pokemon Data. Pokemon has always been a crucial part of my childhood, whether it be the anime or the games, the different types of pokemon, their stats and their legendary status always intrigued me. Here we try to have a look at exactly that from an analytical point of view. Lets have a look at the Dataset.

<img src="{{ site.url }}{{ site.baseurl }}/images/pokemon_eda/Data_head.JPG" alt="Data head">
<img src="{{ site.url }}{{ site.baseurl }}/images/pokemon_eda/Data_tail.JPG" alt="Data tail">

# Data pre-preprocessing

After looking at the dataset, it requires a bit of cleaning like:

- Changing the index value, since we have 2 now
- Changing the "Names" for Mega Pokemons
- Changing some column names to remove spaces from them
- Removing NaN values from the Type 2 Column

New index value and Renaming Mega Pokemons

<img src="{{ site.url }}{{ site.baseurl }}/images/pokemon_eda/Rename_code_2.JPG" alt="Cleaning code">

Changing coloumn names for Types of Pokemons

<img src="{{ site.url }}{{ site.baseurl }}/images/pokemon_eda/Rename_code.JPG" alt="Cleaning code">

Filling NaN values in Type 2 columns to be "None"

<img src="{{ site.url }}{{ site.baseurl }}/images/pokemon_eda/fill_na.JPG" alt="Filling NA">

<img src="{{ site.url }}{{ site.baseurl }}/images/pokemon_eda/Clean_data.JPG" alt="Cleaned Data">

Now the data is cleaned and ready for plotting.

# Basic Numeric Statistics

<img src="{{ site.url }}{{ site.baseurl }}/images/pokemon_eda/Pokemon_types.JPG" alt="All types">

As we can see, we have a total of 18 types of Pokemon for 6 Generations. Now lets check out the numeric distribution of Pokemons per type and generation.

<img src="{{ site.url }}{{ site.baseurl }}/images/pokemon_eda/No_Poke_per_type.JPG" alt="Pokes per type">

As we can see, the highest number of pokemons are Water types followed by Normal and then Grass.

<img src="{{ site.url }}{{ site.baseurl }}/images/pokemon_eda/No_Poke_per_s_type.JPG" alt="Pokes per Secondary type">

When it comes to secondary typing most pokemons have no second type and are pure types. Flying is the most common secondary type amongst all followed by Ground.

<img src="{{ site.url }}{{ site.baseurl }}/images/pokemon_eda/No_Poke_per_gen.JPG" alt="Pokes per Generation">

Generation wise, we can see that Gen 1 as the highest number of Pokemons (This includes the Mega Pokemons so don't confuse it with the anime)

# Pokemon Stats analysis

Okay, so now we know how many pokemons are there in each type, each generation and all but what about all the stats? Lets have a closer look at that.

<img src="{{ site.url }}{{ site.baseurl }}/images/pokemon_eda/Total_dist.JPG" alt="Total Distribution">

This histogram doesn't really help with visualizing the total stats of all the pokemon. Lets try a different approach.

<img src="{{ site.url }}{{ site.baseurl }}/images/pokemon_eda/Total_box.JPG" alt="Total Box">

Okay, now we have a better picture. As we can see that all the normal pokemons have their stats almost normally distributed with the median only slightly on the higher side. But when it comes to Legendary Pokemons, they have a highly positively skewed Total Stat meaning they are relatively stronger than Normal pokemons (That's why they are Legendary Pokemons XD)

Moving forward lets see which generation has the strongest Pokemons

<img src="{{ site.url }}{{ site.baseurl }}/images/pokemon_eda/Box_per_gen.JPG" alt="Box per gen">

This is interesting, even for a game almost all the generations have equally distributed pokemons in terms of strength, with Gen 4 only a slightly higher than others (Thanks Arceus, you god)

Moving on, Lets check out which type of Pokemons are the strongest and which are relatively weaker types.

<img src="{{ site.url }}{{ site.baseurl }}/images/pokemon_eda/Box_per_types.JPG" alt="Box per type">

<img src="{{ site.url }}{{ site.baseurl }}/images/pokemon_eda/Total_bar_types.JPG" alt="Bar per type">

As we can see from these 2 plots, Dragon types are literally overpowering all the other types followed by Steel. Meanwhile bug types are relatively the weakest amongst all (Sorry all you bug type fans out there)

Up until now we still havent really explored the entire dataset, we are only comparing the Total Stats of the pokemons. Lets dive in deeper into the individual stats of each pokemon.

Lets see how these stats are correlated with each other

<img src="{{ site.url }}{{ site.baseurl }}/images/pokemon_eda/Corr_table.JPG" alt="Correlation">

<img src="{{ site.url }}{{ site.baseurl }}/images/pokemon_eda/Corr_plot.JPG" alt="Correlation Plot">

This correlation table and plot is mainly to show that the Total Stats of Pokemon mainly depends on their Attack and their Sp. Attack. Basically a top teir pokemon would have highest attack stats.

Lets now analyse the attack stats.

<img src="{{ site.url }}{{ site.baseurl }}/images/pokemon_eda/total_vs_attack.JPG" alt="Attack vs Total Stats">

Here we can see that Attack and Total Stats are highly correlated (obviously). even Attack and Defense for regular pokemons are positively correlated lets check how that looks.

<img src="{{ site.url }}{{ site.baseurl }}/images/pokemon_eda/Attack_vs_defence.JPG" alt="Attack vs Defence">

Here we see a similar trend in terms of correlation but when it comes to legendary pokemons there is a different case: There is no particular correlation there, a legendary pokemon can have a very high attack stat but very low defense or visa versa.

# Conclusions

- Water type pokemons are the most common
- Generation wise all the pokemons are almost similarly distributed when it comes to Total stats
- Dragon type pokemons are the strongest
- Bug type pokemons are weakest
- Total stats are mainly dependant on the Attack Stats of Pokemons
- The Attack stats and Defense stats are also positively correlated except when it comes to Legendary Pokemons

I hope you liked this blog and it brought back some of your childhood memories grinding hours in these pokemon games, unknown from all the data science behind those little pocket monsters.
Until Next Time.

If you liked this blog, do consider [subscribing](https://docs.google.com/forms/d/e/1FAIpQLSebziVJGTIj3BVelLh5n627G6QIP_fJJsk_qKVaYyfU-atrbg/viewform?usp=sf_link) for updates regarding new blog posts. To find out more about us, visit the [about](https://dataphrase.github.io/about/) section. Until next time, happy dataphrasing!
