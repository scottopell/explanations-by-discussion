# Password Authentication
Storing a user's login information is a tricky task. If you do it wrong then
your site will be insecure. In order to understand why the naive solutions are
bad and how we got to the current advice of "Just Use BCrypt", consider the
following discussion.


## Scene 1
- Eve: Hey Alice, check out this new site I made! All you do is register, post
  some cat pictures, and then everybody can share them!
- Alice: That looks cool! What framework did you use for login?
- Eve: Oh I didn't use a framework, I just stored the passwords in a database
  column.
- Alice: Well that's actually not the best idea. What happens if your database
  contents get leaked? Anybody could login as any of your users, and if your
  users re-use passwords across sites, then they need to race to change all
  their passwords on other sites!
- Eve: Oh you're right! I didn't think of that. I'll work up a different
  solution and get back to you tomorrow.

## Scene 2
- Eve: Hey Alice, check out this cool new system I came up with! I thought
  through your scenario some, and decided that the core issue was that what I
  was storing in my database would allow somebody to log in to the system
  directly. Would you say that that's accurate?
- Alice: Yup, I think you got to the core of it. There needs to be a system
  where even if an attacker has your database, they still can't get in.
- Eve: So basically we want to keep these passwords a secret. And keeping
  secrets is a pretty well explored problem; we'll just encrypt the passwords!
- Alice: Hmm, so you would have one secret key that you would use to encrypt
  everybody's passwords, and you'd just store that somewhere else?
- Eve: Yea, so if the database was compromised then nobody would have my user's
  original passwords, but instead they'd just have this encrypted blob.
- Alice: But now you have a single point of failure. If you had a really
  determined attacker, they could crack your encryption then all of your user's
  passwords would be out in the open.
- Eve: Hmm. You're right. This solution depends on the strength of the
  encryption and my ability to keep the secret key safe. I'll see if I can come
  up with something that doesn't have this single point of failure.

## Scene 3
- Eve: I think I got it! So I figured that at the core of this is the idea that
  what you want to store something that you can check their password against,
  but it isn't just their password.
- Alice: I like where you're going with this, tell me more!
- Eve: So, we want an operation that we can apply to each user's password that
  is one-way only. That is, you can go from a password to this representation,
  but given this representation you can't go back to a plain-text password.
- Alice: Sounds good, does this already exist?
- Eve: This does! This is called a hash function. By definition, a plain hash
  function just maps data of any size to a fixed amount of data, called a
  digest. So if you have a really long string of text, you can apply a hash
  function and get something that is only say 10 characters long.
- Alice: This sounds great! So a user logs in, sends you their password as a
  normal string of text, then you run it through one of these hash functions and
  just store the result in your database, right? That way you can just compare
  the digests when they try to login in the future?
- Eve: You got it! So, what do you think? Am I secure now?
- Alice: I think you're getting closer, but what's to stop somebody from just
  going through common passwords and computing the digest for each one? Then
  they could easily map some user's passwords back to the plain text version
  even though you're only storing the digest.
- Eve: Hmm. I guess nothing. I'll do some research and get back to you.

## Scene 4
- Eve: Hey Alice, I figured out how to stop somebody from just precomputing
  hashes for common passwords.
- Alice: Great! How's it work?
- Eve: So. Your attack focuses on two things, that the hashes are the same
  across all users (and any other site that uses the same hash function), and
  that they can be computed pretty easily. Would you say that's right?
- Alice: Yup, it sounds like those are two big issues.
- Eve: So to solve the first issue, it seems like we'd want to use a different
  hash function for each user. However, that's not really feasible. These things
  are developed by skilled cryptographers and we can't generate a new one
  easily, their time is valuable!
- Alice: All true. So basically, even if two users use the same password, you
  want their password to hash to a different value, correct?
- Eve: Exactly. So what I decided to do was to instead append a random value to
  each user's password before hashing it. One of the properties that
  hash functions have is that if you change the input even a tiny bit, the
  output is completely different. So by appending this random value, I can get
  very different outputs for different users, even with the same password.
- Alice: I think I get it. So then you just store this random value alongside
  the password and then you append it to their password every time they try to
  login?
- Eve: Yup! What do you think?
- Alice: Sounds like it solves that problem pretty well! What about the other
  problem you mentioned? If your database got leaked, and an attacker really
  wanted to get my password, they could just do what they did before and try
  every common password, they just would have to put this other string at the
  end of it.
- Eve: Agreed. How I decided to solve this password was to give cryptographers a
  new requirement for their hash functions. Not only will they need to map input
  to a small, fixed size output, and have output be very different for similar
  input, they now need to be able to make it expensive to compute too.
- Alice: What exactly do you mean by expensive to compute?
- Eve: I mean that it will take a long time for modern processors to run the
  hashing function. Normal hash functions are actually optimized to run fast
  because that's a desireable trait in most of their other applications (caches,
  verifying data integrity, etc.) We want to be able to tell the function, in
  order to get this same hash back, we want it to take at least a half a second
  to compute.
- Alice: Half a second? That doesn't seem that long.
- Eve: In our world it isn't, but remember, processors are really fast. With a
  normal hash function, say SHA256, you can get about 65 thousand hashes per
  second. With more specialized hardware, a standard desktop graphics processor,
  you can get up to 600 million hashes per second. To give more context, this
  means that if you used a password that's all letters and less than 7
  characters long, that's about 8 billion potential combinations (2 ^ 67). On
  our graphics processor, this takes **less than 15 seconds** to get all possible
  combinations and crack your password. Now getting back to the speed of the
  hash function, if each hash now takes a half a second to compute, then it will
  now take the same attacker about **127 years** to crack your password.
- Alice: Wow. Point taken. So does this exist?
- Eve: Yup! There are a few functions that have been developed, namely bcrypt,
  PBKDF2, and scrypt. bcrypt and PBKDF2 are exactly what I described, a hash
  function that takes a work factor that makes it take longer for processors to
  run.
- Alice: How about scrypt, what's different about it?
- Eve: It has one more feature that's interesting. Not only can you make it take
  a long time, but you can make it use a lot of memory. This solves a problem
  that exists in the other two which is custom hardware. Because PBKDF2 and
  bcrypt both take a long time computationally, its possible to design a custom
  circuit that can run that algorithm a lot faster that even graphics processor.
- Alice: A custom circuit? Sorry, I'm lost again.
- Eve: Typical processors, both normal CPUs and graphics processors (GPUs), have
  to be very general in what they can do. Because users will use them for every
  computational task they have, which all use different instructions to tell the
  processor what to do. But if you have a fixed algorithm, then only a certain
  sequence of instructions will be run. So hardware designers can optimize a
  processor for those very specific sequences rather than having to make it be
  able to handle all sorts of computations. Does that clear it up?
- Alice: I think so. So you're saying that most processors have to be able to do
  anything, but if you want to just one run computation very quickly, you can
  switch around stuff at a hardware level to make that one computation super
  fast?
- Eve: Exactly! So what scrypt does is it has a work factor not only for how
  long it takes to run, but it has another work factor that you can tune to make
  it take up a lot of *memory*. This is something that you can't optimize for as
  easily in these custom processors, so it protects you from attackers that have
  lots and lots of resources as well.
- Alice: Wow, you really thought this out!
- Eve: Well you know what they say, if you can't protect the cat pictures then
  what good are you?
- Alice: Umm can't say I've ever heard that, but if you say so!
