# Parte 2: Cucumber
**Feature #1: Filter Movie List** 
La estructura del primer escenario `restrict to movies with "PG" or "R" ratings` será la siguiente:
```
Scenario: restrict to movies with "PG" or "R" ratings
  When I check the following ratings: PG, R
  And I uncheck the following ratings: PG-13, G
  When I press "Refresh"
  Then I should see the following movies: "The Incredibles", "The Terminator"
  And I should not see the following movies: "Aladdin", "Chocolat"
```
Ya tenemos definidos los pasos "should (not) see [...]" y "check the [...] checkbox" en `web_steps.rb`. Entonces podemos definir nuevos pasos que tomen como argumentos varios parámetros a la vez y pase estos uno por uno a los pasos predefinidos. Lo realizamos de la siguiente manera:

```ruby
When /I (un)?check the following ratings: (.*)/ do |no, rating_list|
  rating_list.split(", ").each do |rat|
    steps %Q{
      When I #{no}check the "#{rat}" checkbox
    }
  end
end

Then /^I should (not )?see the following movies: (.*)$/ do |no, movie_list|
  movie_list.split(", ").each do |movie|
    steps %Q{
      Then I should #{no}see #{movie}
    }
  end
end
```

La estructura del escenario `all ratins selected` seria:
```
Scenario: all ratings selected
  When I check the following ratings: PG, PG-13, G, R
  Then I should see all the movies
```
De manera análoga al escenario anterior, podemos definir `should see all the movies` como una iteración de varios pasos `should see`. Lo realizamos de la siguiente manera:

```ruby
Then /I should see all the movies/ do
  Movie.all.each do |movie|
    expect(page.body.include? movie.title)
  end
end
```

**Feature #2: Sort Movie List**
En este caso tendremos dos escenarios: ordenar por título y por fecha de estreno. A estos escenarios le damos la forma:

```
Scenario: sort movies alphabetically
  When I follow "Movie Title"
  Then I should see "Aladdin" before "The Terminator"


Scenario: sort movies in increasing order of release date
  When I follow "Release Date"
  Then I should see "Amelie" before "The Help"
```

El paso "I follow" ya esta predefinido, solo tendriamos que definir el paso "Should see [...] before [...]". Lo hacemos de la siguiente manera:

```ruby
Then /I should see "(.*)" before "(.*)"/ do |e1, e2|
  index_e1 = page.body.index(e1)
  index_e2 = page.body.index(e2)
  expect(index_e1 < index_e2)
end
```