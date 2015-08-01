using UnityEngine;
using System.Collections;
using UnityEngine.UI;

public class HorseController : MonoBehaviour {

	//initialization
	private bool idle;
	private float speed;
	private readonly float walkSpeed = 7.0f;
	private readonly float runSpeed = 14.0f;
	private readonly float secretSpeed = 150.0f;
	private readonly float jumpPower = 10.0f;
	private readonly float maxTiltAngle = 55.0f;

	private int currentTime = 0;
	public Text textObject;

	private float tiltAngle;
	private float smoothness = 0.05f;
	private Quaternion rotation;
	private Vector3 movement;

	public Rigidbody horseBody;
	public Animator horseAnim;

	private readonly int idleStateHash = Animator.StringToHash ("Horse_Idle");
	private readonly int walkStateHash = Animator.StringToHash ("Horse_Walk");
	private readonly int runStateHash = Animator.StringToHash("Horse_Run");

	private Vector3 groundNormal;
	private bool isGrounded;
	private float origGroundCheckDist = 0.6f;
	private float groundCheckDist = 0.6f;

	void Start ()
	{
		//did not null component test because they were assigned in this particular instance
		horseBody = GetComponent<Rigidbody> ();
		horseAnim = GetComponent<Animator> ();

		horseAnim.applyRootMotion = false;

		//sets the text to be transparent at the start of the game
		textObject.enabled = true;
		textObject.CrossFadeAlpha(0.0f, 0.0f, true);

		horseBody.constraints = RigidbodyConstraints.FreezeRotationX | 
							RigidbodyConstraints.FreezeRotationY | 
							RigidbodyConstraints.FreezeRotationZ;
	}
	
	void Update ()
	{
		checkIfGrounded ();
		idle = false;
			//using a switch to simplify break commands and clarify true/false separation
			switch (isGrounded) {
			case true:
				//tests if no movement commands are issued (excluding jump)
				if (Input.GetAxis("Horizontal") == 0 && Input.GetAxis("Vertical") == 0) {
					handleIdle ();
					idle = true;
				}
				if (Input.GetKeyDown (KeyCode.Space)) {
					jump ();
					break;
				}
				if (!idle)
				{
					handleGroundMovement ();
				}
				break;
			case false:
				handleAirMovement ();
				break;
			default:
				//it would be strange if this error message was displayed
				print ("Error");
				break;
			}
		//handles Y-axis rotation (due to player input) after other commands are handled
		float moveHorizontal = Input.GetAxis ("Horizontal");
		transform.Rotate (0.0f, moveHorizontal * 2.3f, 0.0f);
	}


	void checkIfGrounded ()
	{
		RaycastHit hitInfo;
		isGrounded = true;
		//sends a raycast down from the horse to test the ground below it at a set distance. returns the raycast "hitInfo"
		if (Physics.Raycast(transform.position + (transform.up * 0.1f), -transform.up, out hitInfo, groundCheckDist))
		{
			//this gets the normal vector of the ground used for orientation and movement calculations
			groundNormal = hitInfo.normal;
			isGrounded = true;
		}
		else
		{
			//if airborne, this will orient the horse to be parallel with the xz-plane
			groundNormal = Vector3.up;
			isGrounded = false;
		}
		//orients the horse after the normal vectors are calculated
		orientHorse();
	}

	void jump ()
	{
		//mostly preserves previous horse velocities and adds a y component for the jump
		horseBody.velocity = new Vector3(horseBody.velocity.x*0.95f, jumpPower, horseBody.velocity.z*0.95f);
		//when travelling upwards, sets the ground check distance to a small value to ensure the horse gets off the ground
		groundCheckDist = 0.01f;
	}

	void handleIdle ()
	{
		horseBody.velocity = Vector3.zero;
		//changes the animation into an idle animation
		horseAnim.CrossFade (idleStateHash, 0.1f);
		idle = true;
	}

	void handleGroundMovement ()
	{
		//resets animation speed
		horseAnim.speed = 1;

		if (Input.GetKey (KeyCode.LeftShift)) {
			//the "J" key enables an extremely fast speed when held. Used for testing and entertainment purposes
			if (Input.GetKey (KeyCode.J))
			{
				speed = secretSpeed;
			}
			else
			{
				speed = runSpeed;
			}
			horseAnim.Play (runStateHash);
		} else {
			speed = walkSpeed;
			horseAnim.Play (walkStateHash);
		}
		//gets the vertical movement command
		float moveVertical = Input.GetAxis ("Vertical");
		
		movement = new Vector3 (0.0f, 0.0f, moveVertical);
		//localizes the movement direction
		movement = transform.InverseTransformDirection (movement);
		//projects the movement onto the plane of the horses orientation
		movement = Vector3.ProjectOnPlane (movement, groundNormal);
		//makes sure the movement is forward or backwards.
		if (moveVertical < 0.0f) {
			movement = Vector3.RotateTowards (movement, transform.forward, 10, 0.0f);
		} else {
			movement = Vector3.RotateTowards (movement, -transform.forward, 10, 0.0f);
		}
		movement = movement.normalized;
		horseBody.velocity = movement * speed;
	}

	void handleAirMovement ()
	{
		//slows animation speed to look like a jump
		horseAnim.speed = 0.4f;
		//adds extra gravity so the horse accelerates at a normal speed but doesn't affect ground movement
		Vector3 extraGravityForce = (Physics.gravity * 3.1f) - Physics.gravity;
		horseBody.AddForce(extraGravityForce);

		groundCheckDist = horseBody.velocity.y < 0 ? origGroundCheckDist : 0.01f;
	}

	//method orients the horse so that the up vector for the horse (transform.up) is aligned with the normal vector of the ground below it
	void orientHorse ()
	{
		//to find the angle of the terrain below the horse
		//uses the formula: a(dot)b = |a||b|cos(theta)
		tiltAngle = Mathf.Acos(Vector3.Dot (groundNormal, Vector3.up));
		tiltAngle *= Mathf.Rad2Deg;

		//orientation only applies if the horse is not already oriented, and the mountain is not too steep
		if (groundNormal != transform.up && tiltAngle <= maxTiltAngle) {
		
			//Uses cross products to find the new forward direction for after the horse is oriented
			Vector3 biNormal = Vector3.Cross (transform.forward, groundNormal);
			Vector3 newDirection = Vector3.Cross (groundNormal, biNormal);

			//uses quaternions to find the correct new rotation of the horse
			rotation = Quaternion.LookRotation(newDirection, groundNormal);

			//rotates using interpolation to ensure a smooth rotation
			transform.rotation = Quaternion.Slerp (transform.rotation, rotation, Time.deltaTime*speed*smoothness);

		}
	}

	//This holds the story lines to be played during the game
	void LateUpdate () {
		currentTime = Mathf.RoundToInt(Time.realtimeSinceStartup);

		switch (currentTime) {
		case 1:
			//When the game time is rounded to 1 second, "Hello World!" is displayed for 3 seconds
			StartCoroutine(displayText("Hello World!", 3));
			break;

		case 5:
			StartCoroutine(displayText("I will not let this fence cage my spirit", 4));
			break;

		case 10:
			StartCoroutine(displayText("A barn... what's in there?", 4));
			break;

		case 20:
			StartCoroutine(displayText("I've never been so free!", 4));
			break;

		case 28:
			StartCoroutine(displayText("I can sprint... I know I can!", 4));
			break;

		case 36:
			StartCoroutine(displayText("This world is so wonderful!", 4));
			break;

		case 43:
			StartCoroutine(displayText("*NEIGHHHHH!*", 3));
			break;

		case 49:
			StartCoroutine(displayText("Why am I on this path...", 4));
			break;

		case 54:
			StartCoroutine(displayText("Why do I stay on it?", 5));
			break;

		case 63:
			StartCoroutine(displayText("It's okay to go off the path, right?", 4));
			break;

		case 70:
			StartCoroutine(displayText("Where am I going?", 4));
			break;

		case 78:
			StartCoroutine(displayText("What's at the end of this path...", 5));
			break;

		case 87:
			StartCoroutine(displayText("It probably leads nowhere.", 4));
			break;

		case 93:
			StartCoroutine(displayText("...but I can't stop following it.", 5));
			break;

		case 102:
			StartCoroutine(displayText("I feel like I'm being controlled...", 5));
			break;

		case 113:
			StartCoroutine(displayText("Is someone out there?", 6));
			break;

		case 130:
			StartCoroutine(displayText("Hello?", 5));
			break;

		case 138:
			StartCoroutine(displayText("Where are you taking me?", 5));
			break;

		case 150:
			StartCoroutine(displayText("I'm trapped in this body...", 4));
			break;

		case 157:
			StartCoroutine(displayText("It's never been like this...", 5));
			break;

		case 170:
			StartCoroutine(displayText("Am I happy...?", 5));
			break;

		case 178:
			StartCoroutine(displayText("I feel like there's no purpose to all this running around...", 5));
			break;

		case 188:
			StartCoroutine(displayText("What's the point of all of this work?", 4));
			break;

		case 196:
			StartCoroutine(displayText("You're very persistent...", 4));
			break;

		case 203:
			StartCoroutine(displayText("Let me go...", 6));
			break;

		case 210:
			StartCoroutine(displayText("I don't want to be here.", 4));
			break;

		case 220:
			StartCoroutine(displayText("Quit now.", 4));
			break;

		case 228:
			StartCoroutine(displayText("I don't like this... these mountains... they're scaring me.", 5));
			break;

		case 236:
			StartCoroutine(displayText("I've never been so... free?", 6));
			break;

		case 245:
			StartCoroutine(displayText("Hello world......?", 6));
			break;

		case 255:
			StartCoroutine(displayText("Goodbye world......", 5));
			break;

		case 275:
			StartCoroutine(displayText("Credits:", 5));
			break;

		case 281:
			StartCoroutine(displayText("Creator/Director/Lead Programmer: Tim Shur", 5));
			break;

		case 288:
			StartCoroutine(displayText("Lead Writer/Assistant Programmer/Producer: Izhan Ansari", 5));
			break;

		case 295:
			StartCoroutine(displayText("Sound Designer/Graphics Artist/Level Design: Kyle Pappas", 5));
			break;

		case 302:
			StartCoroutine(displayText("3D Artist/Editor/Environment Artist: Kevin Huynh", 5));
			break;

		case 309:
			StartCoroutine(displayText("Special Thanks: Quan Bach and Lawrence Fung", 5));
			break;

		case 315:
			StartCoroutine(displayText("Thanks for playing! :)", 6));
			break;

		case 321:
			StartCoroutine(displayText("HorsePlay", 8));
			break;

		case 350:
			StartCoroutine(displayText("Dude, the game ended.", 5));
			break;

		case 357:
			StartCoroutine(displayText("Didn't you see the credits?", 5));
			break;

		case 363:
			StartCoroutine(displayText("Time is money, bud.", 5));
			break;

		case 369:
			StartCoroutine(displayText("Seeya... for real. :)", 5));
			break;

		case 376:
			StartCoroutine(displayText("Thanks... again. :)", 5));
			break;

		default:
			break;
		}
	}

	//method fades in text and after a delay, fades it out
	IEnumerator displayText (string text, float delay)
	{
		textObject.text = text;
		textObject.CrossFadeAlpha (1.0f, 0.8f, true);
		yield return new WaitForSeconds(delay);
		textObject.CrossFadeAlpha (0.0f, 0.8f, true);
	}
}
