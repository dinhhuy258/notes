# Causal Consistency (/ˈkɑː.zəl/)

The easiest way to explain Causal Consistency is with an example:

Suppose you just placed an order on an e-commerce platform. You get a `success` message.

Then you go to the `all orders` page, and you don’t see the one you just placed.

That would be very irritating, right? Did the order actually go through? Did you get charged? Should you order again?

Then, right when you’re on the phone calling the support center, you reload the page, and the order suddenly appears.

So what happened?

**Eventual Consistency Wasn’t Enough**

Here’s one possible scenario.

The initial submission of the order went to a database server that accepts writes – a Primary server. However, when you requested to read all your orders, you were redirected to a Secondary replica that’s still behind the Primary, e.g., your order is still not replicated. At some point, the order gets copied over to the Secondary, and you see it after a page refresh.

You need stronger guarantees that cover causal (happens-before) relations.

**Transitioning to Causal Consistency**

The activities of submitting your order and then reviewing it are causally related. If your Write request (submit the order) successfully went through, your subsequent Read request (read the orders) should return it.

Eventual Consistency doesn’t deal with that.

## Read-your-writes Consistency

![](https://user-images.githubusercontent.com/17776979/268014203-4e344803-2b38-4f77-bc9b-134e9a40ab3f.png)

Read-your-writes consistency is a consistency model that ensures that once a write operation has been performed, any subsequent read operations by the same user or process will always return the updated value.

## Monotonic Reads Consistency

![](https://user-images.githubusercontent.com/17776979/268015330-f4c60d38-edd7-4d5e-bc6a-88c6f7a92504.png)

Monotonic reads consistency is a consistency model that guarantees that if a user or process reads the latest version of a data item, all subsequent reads from that user or process will return at least the same version or a more recent version of the data. Without a Monotonic Reads guarantee, the Client might get a feeling he’s moving back in time.

## Monotonic Writes

On social media, a user may want to make his album private before uploading a new picture.

In this case, there are two Write operations – one for updating the album access policy and one for the photo upload.

Imagine what will happen if these two writes are somehow applied in the opposite order – first, the photo is uploaded, and then the album is made private.

In that case, the uploaded photo will be public for some time which was never the intention.

Even worse, what if the `privacy Write` gets lost forever?

Let’s review such scenarios.

### Multiple Primary Servers

![](https://user-images.githubusercontent.com/17776979/268017836-cea00d83-c762-48df-abe0-0905c1b6c44f.png)

### Rollback During a Failover

![](https://user-images.githubusercontent.com/17776979/268018271-97ada9e8-66f6-4acc-b4af-c15c7063ec6b.png)

## Writes Follow Reads

![](https://user-images.githubusercontent.com/17776979/268019718-a6160a26-b01a-4285-90b7-bc7cd241730b.png)

Imagine you read a blog post, and you go to the comments section.

You read a comment by some Client A – let’s say he’s asking a question.

You know the answer, so you write your own comment in response to the one by Client A.

Now, imagine a scenario when some Client C goes to the comments and sees only yours but not the one by Client A.

That would be pretty confusing. Your comment alone doesn’t make any sense. It’s supposed to answer a question that is now missing.

## References

[Causal Consistency Guarantees – Case Studies](https://vkontech.com/causal-consistency-guarantees-case-studies)
