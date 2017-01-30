---
layout: post
title: "Refactoring components"
categories: draft
---
The situation here is pretty simple. I am going to be refactoring some components
on a page of a website. The structure is pretty much, <Page> and then <section>
<section>
<section>
Except not so simple. The page is a product detail page - the core of an ecommerece
website. Not only is there a lot going on (understatement) in terms of responsiveness
and the many different states that may be associated with a product, there is also
the fact that the <section> components themselves may show up on other parts of the site
and not only this page.

Take for example the component I am looking at right now. <RecommentationsContentSection>
A brief search for this component in my codebase reveals that it is being imported in
an account overview page, an account recommendations page, the product detail page,
the cart, the 404 page, find in store, as well as cms modules pages.

(Took picture of prop structure)
18 props.

hasContainerDiv: what not to do.
But now I have to search through 7 places to make sure that this boolean was not
passed in at any point.

Question: What steps could have been taken to prevent this current situation?
Question: What is the best way to approach this situation given project restraints?
Also ideal vs. pragramatic because that is fun
Recommendation: Using tests to catalogue each time the component is used 
