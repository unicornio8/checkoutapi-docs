# Concepts

Across our components there are a few reoccurring concepts and/or resources that one should be familiar with in order to fully understand what is going on.

In this section we will go over some of them, and describe on a higher level their significance to our components. Specific use cases will be explained in the endpoints' documentation.

## Client

Every client of our components is a partner of Zalando. Entities using our APIs do so in cooperation with Zalando, given that they are selling Zalando articles in their own platform. Be it a mobile app, or website.

## Sales Channel

Sales channel identifies a few things that are of significance to our components. Sales channel tells us which partner is calling us. It also specifies the catalog available to our end customers, which tells us in which country are the customers attempting to make an order.

## Cart

Cart is a resource of our Checkout API. It holds the articles that the customer wishes to buy. It also provides the total value of the cart, summing the individual prices of each article.

## Checkout

Checkout is the resource with which the remaining data required to create an order is set. It sets the billing address, the delivery address, and provides the client with the payment options available to the customer.

## Address

A quite simple and straightforward resource, Addresses can be used for billing or shipping (a.k.a. delivery). When used for delivery an address can also specify a Pick Up Point.

### Pick Up Point

A pick up point is an address that is used for delivery purposes only. Articles shipped to these addresses will be later picked up (hence the name) by the customer. Pick up points are typically belonging to a company and are specific per country.

An example: a Packstation is a pick up point in Germany.

## Address Checks

This resource is an aid to our clients and our customers. It is used to verify an address for its correctness. It can also provide suggestions for a more easy to recognise address. This resource is mostly useful when creating new addresses.

## Order

Also fairly simple, this resource represents the orders of our customers.