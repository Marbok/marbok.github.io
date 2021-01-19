---
layout: post
title: Category theory. Part 3. Products and Coproducts. ADT
categories: [Category theory]
---
If we want to single out a particular object in a category, we can only do this by describing its pattern of relationships with other objects (and itself). These relationships are defined by morphisms.
This kind of characterization of a particular type of object in terms of its relationship with the rest of the universe is called a **universal construction** and is very common in category theory. We specify a certain property and then establish a hierarchy of objects according to how well they model this property. We then pick the “best” model. Best could mean the simplest, the least constrained, the smallest, or the largest, depending on the context.

If we talk about the hierarchy in categories, then it is necessary to emphasize: the initializing object and the terminal object.
The **initial object** is the object that has one and only one morphism going to any object in the category.
The **terminal object** is the object with one and only one morphism coming to it from any object in the category.

The only difference between the initial and terminal objects was the direction of morphisms. It turns out that for any category C we can define the opposite category C<sup>op</sup> just by reversing all the arrows. The opposite category automatically satisfies all the requirements of a category, as long as we simultaneously redefine composition. The constructions in the opposite category are often prefixed with "co".