## Stratery uses
1) Design a Customer Issue Resolution System: Implement the different ways to assign an agent to a given issue, depending on various factors. https://codezym.com/question/3
2) Design an e-commerce website: To implement different ways to order list of items on search page. e.g. by price, rating or popularity.
3) Design a parking lot: Strategy pattern can be used to implement the different strategies for parking the vehicle. https://codezym.com/question/7
4) Design a Game of Chess: Strategy pattern can be used to implement the different moves for different chess pieces, like straight move (made by rook), diagonal move (bishop), 2+1 move (Knight)

5) ## observer 
1) Design a Food Ordering and Rating System like Zomato, Swiggy, Uber Eats: Whenever customer gives rating to their order (1, 2, 3, 4 or 5) then this rating update must be sent to classes which maintain list of top restaurants based on factors like average rating of restaurant, average rating of a particular food item in the restaurant. Observer pattern is used here. https://codezym.com/question/5
2) ** Design a Movie ticket booking system like BookMyShow: Whenever a new show is added for displaying a movie in a cinema hall, then it needs to be updated to class which maintains list of cinemas running a particular movie in a city or to class which maintains list of shows for a particular movie in a cinema hall. This can also be solved using observer pattern

## Factory 
1) 1) Design a Game of Chess: There are 32 different chess pieces like 2 kings (1 white, 1 black),16 pawns (8 black, 8 white), 4 knights (2 white, 2 black) etc. But all chess pieces have the same move() method. Hence, we use ChessPiece factory here to create all the different pieces.
  
   2) Benefit of using factory pattern is that, later if there is a change in object’s creation logic like a new parameter is added. Then in that case we only need to change code in factory class, else we would have to change code everywhere the object was initialized.
## singleton
But making a class singleton will make it harder to unit test, because singleton object’s instance state is fixed and if one unit test changes the state of singleton object then it may affect other unit tests.
