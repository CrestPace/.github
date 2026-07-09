Landing Page explaining what the product is in simple terms.
  The first thing the user should encounter is our landing page and details of what the product is and also a potential how to use the product section/page. Which the user can access via scrolling down or a CTA(Call to action) button/element.

Account creation page:
  The user should be redirected to the create account page upon clicking a "Getting Started" or "Create Account" CTA button.

Account creation inputs:
  We choose to have only one OAUTH method (Google Auth) for account creation this enables:
    1. Simplicity.
    2. It prevents spamming account creation.

Identity Verification/KYC:
  As already stated in previous docs this will be built from scratch.
  I previously wanted to make users upload their ID then use the liveliness check to match the face against the picture in the ID.

  However, since we are not an official Neo bank and just for the engineering feat. This is a massive privacy breach. So I'll be using an open source self-hosted solution for the matching and avoid interaction with an external LLM to avoid any exposure of user data. The images will be stored in cache with a TTL of 30 mins to allow the user to come back and proceed or... the time taken for the KYC to be completed, which will definitely be less than 30 minutes. Ideally it should be done within 10 minutes. So I'll be optimizing for that.

  After the verification is succesfull, the user will be asked to input username. Here I'll be implementing a bloom filter + DB check. I'm not using any cache as I don't intend for any data stored in cache to persist for more than 2 hours. 

  Username must be unique can be used later for in-app transfers as alternative to account numbers.

  ALso since the docs aren't stored the verification will be stored as any of the 3 states/stages: 
  [done or failed or pending]. This makes it easier later on to write ocmprehensive seed scripts to seed users and test the application without having to go through the entire onboarding flow. 

Token Allocation: 
  After the user has been verified, the user will be asked some slightly personal quetions:
    1. Industry the user currently works in.
    2. Exact role the user has.
    3. City the user works from.

  These will be used to fetch the average salary of that exact role in the city. Then a specified percentage(User specified) will be used as the total amount to be deposited. The aim to to immitate real life interactions.

  After that the user will be asked to choose the percentage to allocate per token. We will start with 10 tokens offering a mix of variety and ease of deploying and development. Though the architecture will be such that we can easily allow usage of up to 100 tokens without hiccups.

  If no token is selected it will default to USDC I don't want to be managing virtual money.

  This 10 tokens also helps for the architecture I designed for the swap logic.

Stop Losses and Locking funds:
  Stop losses was a feature that I wanted to implement since the beginning planning phases of the project. However locking funds was not.

  Also important note: User's lock funds before allocation. 
  
  We planned to allow user to create a savings account and set the period they want to save it for. Then get the interest after the time has been reached. 
  
  We also planned to allow the user to unlock the funds either for full withdrawal or partial withdrawal. 
  
  The architecture we discussed then was to invalidate the interest on full withdrawal before deadline.. then for partial pay based on whats left since in the end what was left was still part of the previou whole so it should be a bit fair. 
  
  But now i decided we will calculate how much the user gained each day for the amount deposited. So that, on full withdrawal the user is getting the principal+ interest that was already being added daily instead of just:
  [If the money stayed there for up to 30 days, then add this amount logic ]
  
  Now this architecture fails whne the user triggers partial withdrawal. Reason: we can't keep giving the same interest when: The amount saved(principal) is lower, since we kept auto adding the interest then we will need to recalculate the interest with the new balance.

  This architecture is also simple cos we don't need any complex logic for partial withdrawals.

  Reminds me that Legacy Systems exists for a reason. No need to reinvent the wheel everytime.
  Sure, maybe we would have came up with an amazing new algorithm that only workded for niche use cases. But this is just more efficient... and effective.

  Now back to how this relates with stop loss, when I came up with my swap feature algo(will talk about in a blog) it had some problems and in a bid to get solutions I thought of this: Locking parts of user funds as a requirement to activate the stop loss setting: Which converts all your tokens to stable coins during a market crash. This by the way brings another problem which ofcourse is a bit easy to solve by ensuring supply is available: I'll explain more later.

  So basically users have to deposit a part of the funds every month (non-recoverable). But I guess since we want to have the customer's at heart we will refund it if they can prove they incurred losses despite our stop loss mechanism. But only the amount for that month will be refunded. The user can choose to opt out that way we don't charge for the stop loss feature. 
  Failure to opt out, then the monthly billing continues.

  This is one of the features that will be tested to ensure correctness and handle possible failures.

User Dashboard:
  Here, the user can view and interact with his/her dashboard.
  See the total amount in USD
  View the total value for each token owned in USD.
  View BNB wallet address
  View Account Number(Only used for in app transfers).
  We won't implement any virtual card feature cos its quite useless.
  Transaction History.
  Buttons and different elements that allow the user to use several app features.


  