# video-game
Side-scrolling video game implemented with Java
import java.awt.event.KeyEvent;
import java.util.Arrays;

public class ZackCeme
{
  
  // set to false to use your code, true will run in DEMO mode (using MattGame.class
  private static final boolean DEMO = false;           
  //public MattGame dg;
  
  // Game window should be wider than tall:   H_DIM < W_DIM   
  private static final int H_DIM = 5;   // # of cells vertically by default: height of game
  private static final int W_DIM = 10;  // # of cells horizontally by default: width of game
  private static final int U_ROW = 0;
  
  private Grid grid;
  private int userRow;
  private int msElapsed;
  private int timesGet;
  private int timesAvoid;
  private int[] prev_col;
  private boolean isPaused;
  private int artifacts;
  private int ammo;
  private int rocksDestroyed;
  
  
  private int pauseTime = 100;
  
  public ZackCeme()
  {

    init(H_DIM, W_DIM, U_ROW);
  }
  
  public ZackCeme(int hdim, int wdim, int uRow)
  {
    init(hdim, wdim,uRow);

  }
  
  private void init(int hdim, int wdim, int uRow) {  
    grid = new Grid("space.png", hdim, wdim);   
    
    //look at the various Grid constructors to see what else you can do!
    //Example:
    //grid = new Grid(hdim, wdim, Color.MAGENTA);   
    ///////////////////////////////////////////////
    userRow = uRow;
    msElapsed = 0;
    timesGet = 0;
    timesAvoid = 5;
    artifacts = 0;
    ammo = 10;
    rocksDestroyed = 0;
    prev_col = new int[hdim];
    Arrays.fill(prev_col, 1);
    updateTitle();
    isPaused = false;
    grid.setImage(new Location(userRow, 0), "ufo.gif");
    
  }
  
  public void play()
  {
    
    while (!isGameOver())
    {
      grid.pause(pauseTime);//the game pauses for a certain amount of time indicated by the variable pauseTime
      handleKeyPress();//this handles the keyboard presses
      while (isPaused)
      {
        grid.pause(pauseTime);
        handlePause();
      }
        if (msElapsed % (3 * pauseTime) == 0)//every 3rd pause, this will bring in any number of A's and G's
        {
          scrollLeft();
          populateRightEdge();                     
        }
        updateTitle();//this updates the score
        msElapsed += pauseTime;
      }
    }
  
  public void handleKeyPress()
  {
    int key = grid.checkLastKeyPressed();
   // isPaused = false;
    
    if (key == KeyEvent.VK_UP)//gonna have to call handleCollision here
    {
      grid.setImage(new Location(userRow, 0), null);
      if (userRow == 0)
        grid.setImage(new Location(userRow, 0), "ufo.gif");
      else
      {
        userRow--;
        grid.setImage(new Location(userRow, 0), "ufo.gif");
        handleCollision(new Location(userRow, 0));
      }
    }
    if (key == KeyEvent.VK_DOWN)//gonna have to call handleCollision here
    {
      grid.setImage(new Location(userRow, 0), null);
      if (userRow == H_DIM-1)
        grid.setImage(new Location(userRow, 0), "ufo.gif");
      else
      {
        userRow++;
        grid.setImage(new Location(userRow, 0), "ufo.gif");
        handleCollision(new Location(userRow, 0));
      }
    }
    //http://docs.oracle.com/javase/7/docs/api/constant-values.html#java.awt.event.KeyEvent.VK_DOWN
    if (key == KeyEvent.VK_Q)
      System.exit(0);
    
    // use the 'T' key to help you with implementing speed up/slow down/pause
    else if (key == KeyEvent.VK_T) 
    {
      boolean interval = (msElapsed % (3 * pauseTime) == 0);
      System.out.println("pauseTime " + pauseTime + " msElapsed reset " + msElapsed 
                        + " interval " + interval);
    }
    if (key == KeyEvent.VK_COMMA)
    {
      if (pauseTime > 10)
      {
        pauseTime *= 0.9;
        msElapsed = 0;
      }
    }
    if (key == KeyEvent.VK_PERIOD)
    {
      if (pauseTime < 214)
      {
        pauseTime *= 1.1;
        msElapsed = 0;
      }
    }
    if (key == KeyEvent.VK_P)
    {
      isPaused = true;
     }
    if (key == KeyEvent.VK_SPACE)
    {
      
      if (ammo != 0 && artifacts < 10)
      {
        grid.setImage(new Location(userRow, 1), "lazerbeam.gif");
        grid.pause(pauseTime);
        lazerCollision(new Location(userRow, 1));
        ammo--;
      }
      else if (ammo != 0 && artifacts >= 10)
      {
        pauseTime = 80;
        msElapsed = 0;
        grid.setImage(new Location(userRow, 1), "lazer2extra.gif");
        grid.pause(pauseTime);
        grid.setImage(new Location(userRow, 2), "lazerbeam2.gif");
        grid.pause(pauseTime);
        lazerCollision(new Location(userRow, 2));
        ammo--;
      }
    }
    if (key == KeyEvent.VK_C)
    {
      if (ammo == 0 && artifacts < 10)
        ammo = 10;
      if (ammo == 0 && artifacts >= 10)
        ammo = 15;
    }
  }
  public void handlePause()
  {
    int key = grid.checkLastKeyPressed();
    if (key == KeyEvent.VK_P)
    {
      isPaused = false;
    }
  }
  
  public void populateRightEdge()
  {
    int[] curr_col = new int[H_DIM];
    do {
    int right_row = 0;
    while (right_row < H_DIM)
    {
      double x = Math.random();
      if (x < 0.005)
      {
        grid.setImage(new Location(right_row, W_DIM-1), "artifact.gif");
        curr_col[right_row] = 1;
      }
      else if (x < 0.10)
      {
        grid.setImage(new Location(right_row, W_DIM-1), "get.gif");
        curr_col[right_row] = 1;
      }
      else if (x < 0.35)
      {
        grid.setImage(new Location(right_row, W_DIM-1), "avoid.gif");
        curr_col[right_row] = 0;
      }
      else
      {
        grid.setImage(new Location(right_row, W_DIM-1), null);
        curr_col[right_row] = 1;
      }
      right_row++;
    }
    } while (unwinnable(curr_col));
    prev_col = curr_col;
  }
  
  private boolean unwinnable(int[] curr_col)
  {
    for (int i = 0; i < curr_col.length; i++)
    {
      if (curr_col[i] == 1 && prev_col[i] == 1)
        return false;
    }
    return true;
  }
  
  public void scrollLeft()
  {
    for (int i = 1; i < W_DIM; i++)
    {
      oneLeft(i);
    }
  }
  private void oneLeft(int i)
  {
    String image_name;
    for (int x = 0; x < H_DIM; x++)
    {
      image_name = grid.getImage(new Location(x, i-1));
      if (image_name != null && image_name.equals("lazerbeam.gif")){
        lazerCollision(new Location(x, i));//if i press space right in front of a rock, it'll give me a point
        grid.setImage(new Location(x,i-1), null);
      }
      else if (image_name != null && image_name.equals("lazerbeam2.gif")){
        lazerCollision(new Location(x, i));
        grid.setImage(new Location(x,i-1), null);
      }
      if(false){
      }
      else{
        if (image_name != null && image_name.equals("ufo.gif")) {
        handleCollision(new Location(x, i));
        }
        else
        {
          grid.setImage(new Location(x, i-1), grid.getImage(new Location(x, i)));
        }
      }
    }
  }
  public void handleCollision(Location loc)
  {
    String img = grid.getImage(loc);
    if (img == null) { }      
    else if (img.equals("artifact.gif"))
      artifacts++;
    else if (img.equals("get.gif"))
    {
      if (timesGet <= 50)
        timesGet++;
      if (timesGet > 50)
        timesGet += 2;
      if (timesGet % 25 == 0)
      {
        timesAvoid++;
      }
    }
    else if (img.equals("avoid.gif"))
      timesAvoid--;
  }
  public void lazerCollision(Location loc)
  {
    String img = grid.getImage(loc);
    if (img == null) {}
    else if (img.equals("avoid.gif"))
    {
      rocksDestroyed++;
      grid.setImage(loc, "pebbles.gif");
    }
  }
  
  public int getScore()
  {
    return artifacts;
  }
  public int livesRemain()
  {
    return timesAvoid;
  }
  
  public void updateTitle()
  {
    grid.setTitle("Artifacts:  " + getScore() + " | " + "Lives: " + livesRemain() + 
                  " | rounds: " + ammo + " | smashed rocks: " + rocksDestroyed);
    if (timesAvoid == 0)
      grid.setTitle("Game Over! You destroyed: " + rocksDestroyed + " rocks");
  }
     
  public boolean isGameOver()
  {
    if (timesAvoid == 0)
      return true;
    return false;
  }
  
  public static void test()
  {
    if (DEMO) {       // reference game: 
                      //   - start by playing the demo version to get a handle on what 
                      //   - go back to the demo anytime you don't know what your next step is
                      //     or details about it are not concrete
                      //         figure out according to the game play 
                      //         (the sequence of display and action) how the functionality
                      //         you are implementing next is supposed to operate
                      // It's critical to have a plan for each piece of code: follow, understand
                      // and study the assignment description details; and explore the basic game. 
                      // You should always know what you are doing (your current, small goal) before
                      // implementing that piece or talk to us. 

      System.out.println("Running the demo: DEMO=" + DEMO);
      //default constructor   (4 by 10)
      //MattGame game = new MattGame();
      // try the two parameter constructor with MattGame
      // MattGame game = new MattGame(10, 20, 0);
      //game.play();
    
    } else {
      System.out.println("Running student game: DEMO=" + DEMO);
      //This code runs when DEMO is false!
      
      //test 1: with parameterless constructor
      ZackCeme game = new ZackCeme();
      
      //test 2: with constructor specifying grid size, you need to implement this!
      //it should work as long as height < width
      //Game game = new Game(10, 20, 4);
      
      game.play();
    }
  }
  
  public static void main(String[] args)
  {
    test();
  }
}
