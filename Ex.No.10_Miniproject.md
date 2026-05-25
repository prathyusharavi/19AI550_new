# Ex.No: 10 | Implementation of 2D Mini-Project (Football Arcade Game)
### DATE: 25-06-2026                                                                            
### REGISTER NUMBER : 212223240187

## AIM: 
To design, develop, and deploy a fully functional 2D Football Arcade Game mini-project in Unity, utilizing custom C# architectures, state-driven predictive striking AI, interactive user pre-match inputs, and rigorous physics boundary constraint handlings.

## ALGORITHM:
1. **Workspace Matrix Initializations:** Initialize a 2D Unity scene grid. Assign high-fidelity field background textures and configure visual boundaries using specialized layout methods. Set custom sprite assets with precise tight mesh profiles.
2. **Kinematic Grid & Contact Management:** Apply `Circle Collider 2D` and `Rigidbody2D` physical assets to the ball object, switching simulation checking parameters to `Continuous` to mitigate boundary tunneling. Map borders using zero-friction `Physics Material 2D` setups to eliminate frame scraping.
3. **State Machine Management & Watchdogs:** Deploy a centralized `GameManager` loop. Attach callback listeners to individual goal triggers. Set an independent outer field boundary boundary tracker to catch and center out-of-bounds assets within 2 seconds.
4. **Behavioral AI Subroutine Mapping:** Implement dynamic speed shifts inside the AI controller script to alternate tracking responsiveness based on immediate vector data. Calculate real-time targeting vectors utilizing linear projection math between the ball's center and the target goal boundary coordinates.
5. **Exception Handling Routines:** Implement a distance-buffer checkpoint loop inside the physics step handler. If proximity metrics detect localized compression locks near corners, clear the tracking vectors, apply immediate micro-retreat forces, and impart an artificial outward velocity bounce vector to return assets to active play.

---

## PROGRAM SCRIPTS:

### 1. GameManager.cs
```csharp
using UnityEngine;
using System.Collections;
using TMPro;
using UnityEngine.UI;

public class GameManager : MonoBehaviour
{
    public static GameManager instance;
    public BallMovement ball;
    
    [Header("UI Text Elements")]
    public TextMeshProUGUI player1Text; 
    public TextMeshProUGUI player2Text;
    public TextMeshProUGUI timerText;    
    public TextMeshProUGUI winStatusText;

    [Header("Opening Coin Flip Buttons")]
    public Button headsButton;
    public Button tailsButton;

    private int player1Score = 0;
    private int player2Score = 0;
    private float timeRemaining = 60f; 
    private bool gameActive = false; 
    [HideInInspector] public bool isHandlingGoal = false; 
    private int lastScorer = 0; 

    private float outOfBoundsTimer = 0f;
    private float fieldLimitX = 8.5f; 
    private float fieldLimitY = 4.8f; 

    void Awake()
    {
        if (instance == null) instance = this;
        else Destroy(gameObject);
    }

    void Start()
    {
        if (ball != null) ball.GetComponent<Rigidbody2D>().linearVelocity = Vector2.zero;
        winStatusText.text = "CHOOSE HEADS OR TAILS TO KICKOFF!";
        timerText.text = "1:00";

        headsButton.onClick.AddListener(() => UserSelectedCoinSide(0)); 
        tailsButton.onClick.AddListener(() => UserSelectedCoinSide(1)); 
    }

    void Update()
    {
        if (!gameActive || isHandlingGoal) return;

        if (timeRemaining > 0)
        {
            timeRemaining -= Time.deltaTime;
            DisplayTime(timeRemaining);
        }
        else
        {
            timeRemaining = 0;
            EndMatch();
        }
        CheckBallBoundaries();
    }

    void UserSelectedCoinSide(int userChoice)
    {
        headsButton.gameObject.SetActive(false);
        tailsButton.gameObject.SetActive(false);
        StartCoroutine(OpeningMatchCoinFlip(userChoice));
    }

    IEnumerator OpeningMatchCoinFlip(int userChoice)
    {
        winStatusText.text = "FLIPPING COIN...";
        yield return new WaitForSeconds(1.5f);

        int coinResult = Random.Range(0, 2); 
        string resultText = (coinResult == 0) ? "HEADS" : "TAILS";
        Rigidbody2D ballRb = ball.GetComponent<Rigidbody2D>();

        if (userChoice == coinResult)
        {
            winStatusText.text = resultText + "! YOU WIN KICKOFF!";
            yield return new WaitForSeconds(1.2f);
            winStatusText.text = "READY...";
            yield return new WaitForSeconds(0.6f);
            winStatusText.text = "";
            gameActive = true; 
            if (ballRb != null) ballRb.linearVelocity = new Vector2(-ball.baseSpeed, 0f); 
        }
        else
        {
            winStatusText.text = resultText + "! AI WINS KICKOFF!";
            yield return new WaitForSeconds(1.2f);
            winStatusText.text = "READY...";
            yield return new WaitForSeconds(0.6f);
            winStatusText.text = "";
            gameActive = true; 
            if (ballRb != null) ballRb.linearVelocity = new Vector2(ball.baseSpeed, 0f); 
        }
    }

    public void BallEnteredGoal(bool isLeftGoal)
    {
        if (!gameActive || isHandlingGoal) return;
        isHandlingGoal = true; 

        Rigidbody2D ballRb = ball.GetComponent<Rigidbody2D>();
        if (ballRb != null) ballRb.linearVelocity = Vector2.zero;

        if (isLeftGoal)
        {
            player2Score++;
            player2Text.text = player2Score.ToString();
            lastScorer = 2; 
            winStatusText.text = "GOAL FOR AI!";
        }
        else
        {
            player1Score++;
            player1Text.text = player1Score.ToString();
            lastScorer = 1; 
            winStatusText.text = "YOU SCORE!!";
        }
        StartCoroutine(StandardGoalResetSequence());
    }

    IEnumerator StandardGoalResetSequence()
    {
        yield return new WaitForSeconds(2f);
        if (!gameActive) yield break;

        winStatusText.text = "READY...";
        ball.transform.position = Vector2.zero;
        Rigidbody2D ballRb = ball.GetComponent<Rigidbody2D>();
        if (ballRb != null) ballRb.linearVelocity = Vector2.zero;
        
        yield return new WaitForSeconds(0.8f);
        winStatusText.text = ""; 
        isHandlingGoal = false; 
        
        float launchX = (lastScorer == 1) ? 1f : -1f; 
        float launchY = Random.Range(-0.4f, 0.4f);
        Vector2 directionalVector = new Vector2(launchX, launchY).normalized;
        if (ballRb != null) ballRb.linearVelocity = directionalVector * ball.baseSpeed;
    }

    void CheckBallBoundaries()
    {
        if (ball == null || isHandlingGoal) return;
        bool isOutsideX = Mathf.Abs(ball.transform.position.x) > fieldLimitX;
        bool isOutsideY = Mathf.Abs(ball.transform.position.y) > fieldLimitY;

        if (isOutsideX || isOutsideY)
        {
            outOfBoundsTimer += Time.deltaTime;
            if (outOfBoundsTimer >= 2.0f) 
            {
                outOfBoundsTimer = 0f;
                ResetRound();
            }
        }
        else outOfBoundsTimer = 0f;
    }

    void DisplayTime(float timeToDisplay)
    {
        if (timeToDisplay < 0) timeToDisplay = 0;
        float minutes = Mathf.FloorToInt(timeToDisplay / 60); 
        float seconds = Mathf.FloorToInt(timeToDisplay % 60);
        timerText.text = string.Format("{0:0}:{1:00}", minutes, seconds);
    }

    void ResetRound()
    {
        if (ball != null)
        {
            ball.transform.position = Vector2.zero;
            ball.GetComponent<Rigidbody2D>().linearVelocity = Vector2.zero;
        }
        outOfBoundsTimer = 0f;
        isHandlingGoal = false;
        winStatusText.text = "";
        if (gameActive) Invoke("LaunchBallAgain", 1f);
    }

    void LaunchBallAgain()
    {
        if (gameActive && ball != null && !isHandlingGoal) ball.SendMessage("LaunchBall");
    }

    void EndMatch()
    {
        gameActive = false;
        isHandlingGoal = true; 
        StopAllCoroutines();
        timerText.text = "0:00";

        if (ball != null)
        {
            ball.transform.position = Vector2.zero;
            ball.GetComponent<Rigidbody2D>().linearVelocity = Vector2.zero; 
        }



        if (player1Score > player2Score) winStatusText.text = "YOU WIN!";
        else if (player2Score > player1Score) winStatusText.text = "AI WINS!";
        else winStatusText.text = "IT'S A DRAW!";
    }
}
```
### 2. AIPaddle.cs
```csharp
using UnityEngine;

public class AIPaddle : MonoBehaviour
{
    [Header("Speed Settings")]
    public float defenseSpeed = 3.5f; 
    public float attackSpeed = 12.0f;  
    
    [Header("Human Error Settings")]
    public float reactionDelay = 0.18f; 
    
    [Header("Target Goal Coordinates")]
    private float targetGoalX = -8.1f;     
    private float targetGoalMinY = -2.3f;  
    private float targetGoalMaxY = 2.3f;   

    public Rigidbody2D rb;
    private GameObject ball;
    private Rigidbody2D ballRb;
    private GameObject playerPaddle; 

    private float minX = 4.0f;  
    private float maxX = 7.0f;  
    private float minY = -3.5f; 
    private float maxY = 3.5f;

    private float stuckTimer = 0f;
    private bool isBackingAway = false;
    private float backAwayDuration = 0.8f; 
    private float backAwayCountdown = 0f;
    private Vector2 delayedBallPosition;
    private float delayTimer = 0f;

    void Start()
    {
        ball = GameObject.Find("Ball");
        if (ball != null) ballRb = ball.GetComponent<Rigidbody2D>();
        playerPaddle = GameObject.Find("LeftPaddle");
        if (ball != null) delayedBallPosition = ball.transform.position;
    }

    void FixedUpdate()
    {
        if (ball == null || ballRb == null) return;

        if (GameManager.instance != null && GameManager.instance.isHandlingGoal)
        {
            Vector2 centralAIPosition = new Vector2(6.0f, 0f); 
            rb.MovePosition(Vector2.MoveTowards(rb.position, centralAIPosition, attackSpeed * Time.fixedDeltaTime));
            stuckTimer = 0f;
            isBackingAway = false;
            return; 
        }

        if (isBackingAway)
        {
            backAwayCountdown -= Time.fixedDeltaTime;
            Vector2 escapePoint = new Vector2(6.5f, rb.position.y);
            rb.MovePosition(Vector2.MoveTowards(rb.position, escapePoint, attackSpeed * Time.fixedDeltaTime));

            if (backAwayCountdown <= 0f)
            {
                isBackingAway = false; 
                stuckTimer = 0f;
            }
            return;
        }

        float distanceToBall = Vector2.Distance(transform.position, ball.transform.position);
        bool ballInCornerY = Mathf.Abs(ball.transform.position.y) > 3.2f;
        bool ballInDeepX = ball.transform.position.x > 6.8f;

        if ((ballInCornerY && ballInDeepX && distanceToBall < 1.1f) || (ballRb.linearVelocity.magnitude < 0.4f && distanceToBall < 0.9f))
        {
            isBackingAway = true;
            backAwayCountdown = backAwayDuration;

            Vector2 normalVector = (ball.transform.position.y > 0) ? Vector2.down : Vector2.up;
            Vector2 exitVector = (Vector2.left + normalVector * 0.5f).normalized;
            
            BallMovement ballScript = ball.GetComponent<BallMovement>();
            float kickSpeed = (ballScript != null) ? ballScript.baseSpeed : 7f;
            ballRb.linearVelocity = exitVector * kickSpeed;
            return;
        }
        else stuckTimer = Mathf.MoveTowards(stuckTimer, 0f, Time.fixedDeltaTime * 2f);

        delayTimer += Time.fixedDeltaTime;
        if (delayTimer >= reactionDelay)
        {
            delayedBallPosition = ball.transform.position; 
            delayTimer = 0f;
        }

        Vector2 targetPosition;
        float currentMovementSpeed;

        if (ballRb.linearVelocity.x > 0)
        {
            currentMovementSpeed = defenseSpeed; 
            targetPosition = new Vector2(6.5f, delayedBallPosition.y);
        }
        else if (ball.transform.position.x > 0)
        {
            currentMovementSpeed = attackSpeed;
            float calculatedTargetY = 0f;
            if (playerPaddle != null)
            {
                if (playerPaddle.transform.position.y > 0.5f) calculatedTargetY = targetGoalMinY; 
                else if (playerPaddle.transform.position.y < -0.5f) calculatedTargetY = targetGoalMaxY; 
                else calculatedTargetY = (ball.transform.position.y > 0) ? targetGoalMaxY : targetGoalMinY;
            }

            Vector2 strategicGoalTarget = new Vector2(targetGoalX, calculatedTargetY);
            Vector2 vectorToBall = (delayedBallPosition - strategicGoalTarget).normalized;

            if (distanceToBall < 1.0f) targetPosition = new Vector2(rb.position.x - 0.2f, delayedBallPosition.y);
            else targetPosition = delayedBallPosition + (vectorToBall * 0.75f); 
        }
        else
        {
            currentMovementSpeed = defenseSpeed;
            targetPosition = new Vector2(5.5f, delayedBallPosition.y * 0.3f); 
        }

        targetPosition.x = Mathf.Clamp(targetPosition.x, minX, maxX);
        targetPosition.y = Mathf.Clamp(targetPosition.y, minY, maxY);
        rb.MovePosition(Vector2.MoveTowards(rb.position, targetPosition, currentMovementSpeed * Time.fixedDeltaTime));
    }
}
```
### 3. BallMovement.cs
```csharp
using UnityEngine;

public class BallMovement : MonoBehaviour
{
    public float baseSpeed = 7f;      
    public float wallSpeedFactor = 0.8f; 
    private float currentSpeed;
    public Rigidbody2D rb;

    void Start()
    {
        currentSpeed = baseSpeed;
        LaunchBall();
    }

    void LaunchBall()
    {
        currentSpeed = baseSpeed;
        float xDir = Random.Range(0, 2) == 0 ? -1f : 1f;
        float yDir = Random.Range(-0.5f, 0.5f);
        Vector2 launchDirection = new Vector2(xDir, yDir).normalized;
        rb.linearVelocity = launchDirection * currentSpeed;
    }

    void OnCollisionEnter2D(Collision2D collision)
    {
        Vector2 incomingVector = rb.linearVelocity;
        Vector2 normalVector = collision.contacts[0].normal; 
        Vector2 reflectDirection = Vector2.Reflect(incomingVector.normalized, normalVector);

        if (collision.gameObject.name.Contains("Paddle"))
        {
            currentSpeed = baseSpeed;
            rb.linearVelocity = reflectDirection.normalized * currentSpeed;
        }
        else if (collision.gameObject.name.ToLower().Contains("wall"))
        {
            currentSpeed = baseSpeed * wallSpeedFactor; 
            if (Vector2.Dot(reflectDirection.normalized, normalVector) < 0.4f)
            {
                reflectDirection = (reflectDirection.normalized + normalVector * 0.5f).normalized;
            }
            rb.linearVelocity = reflectDirection.normalized * currentSpeed;
            transform.position = (Vector2)transform.position + (normalVector * 0.08f);
        }
    }

    void FixedUpdate()
    {
        if (rb.linearVelocity != Vector2.zero && !Mathf.Approximately(rb.linearVelocity.magnitude, currentSpeed))
        {
            rb.linearVelocity = rb.linearVelocity.normalized * currentSpeed;
        }
    }
}
```

### Output:
### 1.Start
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/b955f2aa-6e7a-4093-a7ab-77bf54ac380b" />

### 2. Toss
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/5a37ba00-6701-4da3-b47f-06cbafc67f30" />

### 3. Gameplay
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/41c0cd75-e104-4654-b0b8-5125d4aef24c" />
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d74c3cb5-9fa7-4698-a2f1-4138e575a5cf" />

### Result: Thus, the 2D Football Arcade mini-project was designed, programmatically implemented, debugged for high-speed edge collisions, and pushed successfully into the upstream version control tree via Git tracking protocols.





