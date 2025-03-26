https://medium.com/@prashant558908/design-of-a-food-ordering-system-like-zomato-swiggy-doordash-in-both-java-and-python-for-low-7d178a5f7b4c

## Requirements:
You basically do 3 things in a food ordering system

1. Browse list of restaurants sorted by rating, price, popularity etc.
Above problem statement specifies two ways of listing restaurants

getTopRatedRestaurants()
getTopRestaurantsByFood(String foodItemId)
2. Order food item from restaurants

an orderId will already be given and you won’t need to generate one

orderFood(String orderId, String restaurantId, String foodItemId)
3. Rate your order

You can assign a rating of 1, 2, 3, 4 or 5 stars to your order. 5 stars is the beast rating and 1 means the worst rating.

when you are giving rating an order e.g giving 4 stars to an order, then it means you are assigning 4 stars to both the food item in that restaurant as well as 4 stars to the overall restaurant rating.

rateOrder(String orderId, int rating)

## order
class Order{
    private String orderId, restaurantId, foodItemId;
    private int rating;
}

## Rating
Also we need to keep track of overall average rating for restaurants and food items. Lets have a class Rating for that. This class keep track of sum of all ratings received and number of people who have assigned rating.

e.g. if 4 people have rated food items from a particular restaurant as 4, 3, 5 and 2 then its sum will be 4+3+5+2 = 14 and count will be 4, so overall average rating of the restaurant will be 14/4 = 3.5

class Rating{
    private int sum=0, count=0;

    Rating(int sum, int count){
         this.sum=sum;
         this.count=count;
     }

    // rating is rounded down to one decomal point
    double getAverageRating(){
        if(count<=0) return 0;
        double rating = (double)sum/count;
        rating = (double)((int)((rating+0.05)*10))/10.0;
        return rating;
    }

    void add(int num){
        this.sum+=num;
        this.count+=1;
    }
}

Next we will need a class OrdersManager to track all orders and rate them. It will keep all orders in a map and have methods for ordering food and rating any order.
class OrdersManager {
  // orderId vs Order
    private HashMap<String, Order> map = new HashMap<>();

    void orderFood(String orderId, String restaurantId, String foodItemId){
        Order order = new Order(orderId, restaurantId, foodItemId, 0);
        map.put(orderId, order);
    }

    void rateOrder(String orderId, int rating){
        Order order = map.get(orderId);
        order.setRating(rating);
    }
}
## top n resturant

    // restaurantId vs rating
    private HashMap<String, Rating> ratings = new HashMap<>();
    
    List<String> getRestaurants(int n){}
}

class MostRatedRestaurantsByFood {
    // foodItemId vs restaurantId vs rating
    private HashMap<String, HashMap<String, Rating>> ratings = new HashMap<>();

    List<String> getRestaurants(String foodItemId, int n){}
}

Here is what our Solution class which is sort of our driver or controller class, look like. It will simply use classes we created above to fulfill all its functionalities.

public class Solution implements Q05RestaurantRatingInterface {
    private OrdersManager ordersManager;
    private MostRatedRestaurants mostRatedRestaurants;
    private MostRatedRestaurantsByFood mostRatedRestaurantsByFood;

    public void init(Helper05 helper){
        ordersManager = new OrdersManager();
        mostRatedRestaurants = new MostRatedRestaurants();
        mostRatedRestaurantsByFood= new MostRatedRestaurantsByFood();
    }

    public void orderFood(String orderId, String restaurantId, String foodItemId) {
        ordersManager.orderFood(orderId,restaurantId, foodItemId);
    }

    public void rateOrder(String orderId, int rating) {
        ordersManager.rateOrder(orderId, rating);
    }

    public List<String> getTopRestaurantsByFood(String foodItemId) {
        return mostRatedRestaurantsByFood.getRestaurants(foodItemId,  20);
    }

    public List<String> getTopRatedRestaurants() {
        return mostRatedRestaurants.getRestaurants(20);
    }
}

## Observer Pattern
We will use observer design pattern to send rate order updates from class OrdersManager to or list classes MostRatedRestaurants and class MostRatedRestaurantsByFood .

Now we can achieve our goal by updating these classes directly from OrdersManager class. But that will make restaurants listing classes tightly coupled with the OrdersManager class. i.e. in case we need a new class in future which needs rating updated for orders say for sending notifications or analytics then code in class OrdersManager will also need to be updated and we want to avoid that.

Observer pattern will give us loose coupling between OrdersManager class and restaurants listing classes and it will make it easy to add new observers in futur

Both restaurant listing classes MostRatedRestaurants and MostRatedRestaurantsByFood will be our observers. They will implement the interface RateOrderObserver and update the ratings of restaurants in their own maps whenever the update() method is called

interface RateOrderObserver{
  void update(Order order);
}
class MostRatedRestaurants implements RateOrderObserver{
    // restaurantId vs rating
    private HashMap<String, Rating> ratings = new HashMap<>();

    public  void update(Order order) {
        ratings.putIfAbsent(order.getRestaurantId(), new Rating(0,0));
        Rating rating = ratings.get(order.getRestaurantId());
        rating.add(order.getRating());
        }
}
class MostRatedRestaurantsByFood implements RateOrderObserver{
    // foodItemId vs restaurantId vs rating
    private HashMap<String, HashMap<String, Rating>> ratings = new HashMap<>();

    public  void update(Order order) {
        ratings.putIfAbsent(order.getFoodItemId(), new HashMap<>());
        HashMap<String, Rating> restaurantsMap=ratings.get(order.getFoodItemId());
        restaurantsMap.putIfAbsent(order.getRestaurantId(), new Rating(0,0));
        Rating rating = restaurantsMap.get(order.getRestaurantId());
        rating.add(order.getRating());
    }
}

Class OrdersManager will be the subject. It will keep a list of observers and will notify all of them with notifyAll() whenever any order is rated by the customer i.e. rateOrder() is called.
class OrdersManager {
    private HashMap<String, Order> map = new HashMap<>();
    private ArrayList<RateOrderObserver> observers = new ArrayList<>();

    void rateOrder(String orderId, int rating){
        Order order = map.get(orderId);
        order.setRating(rating);
        notifyAll(order);
    }

    public void addObserver(RateOrderObserver observer) {
        observers.add(observer);
    }

    public void notifyAll(Order order) {
        for(RateOrderObserver observer : observers) observer.update(order);
    }
}

## Main
public class Solution implements Q05RestaurantRatingInterface {
    private Helper05 helper;
    private OrdersManager ordersManager;
    private MostRatedRestaurants mostRatedRestaurants;
    private MostRatedRestaurantsByFood mostRatedRestaurantsByFood;
    
    public void init(Helper05 helper){
        this.helper=helper;
        ordersManager = new OrdersManager();
        mostRatedRestaurants = new MostRatedRestaurants();
        mostRatedRestaurantsByFood= new MostRatedRestaurantsByFood();

        ordersManager.addObserver(mostRatedRestaurants);
        ordersManager.addObserver(mostRatedRestaurantsByFood);
    }
}
