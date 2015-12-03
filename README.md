using UnityEngine;
using System.Collections;
 
public class PlayerController : MonoBehaviour
{
 public float speed = 10, jumpVelocity = 10;
 public LayerMask playerMask;
 public bool canMoveInAir = true;
 Transform myTrans, tagGround;
 Rigidbody2D myBody;
 bool isGrounded = false;
 float hInput;
 AnimatorController myAnim;
 
 void Start ()
 {
//  myBody = this.rigidbody2D;//Unity 4.6-
  myBody = this.GetComponent<Rigidbody2D> ();//Unity 5+
  myTrans = this.transform;
  tagGround = GameObject.Find (this.name + "groundCheck").transform;
  myAnim = AnimatorController.instance;
 }
 void FixedUpdate ()
 {
  isGrounded = Physics2D.Linecast (myTrans.position, tagGround.position, playerMask);
  myAnim.UpdateIsGrounded (isGrounded);
  
  #if !UNITY_ANDROID && !UNITY_IPHONE && !UNITY_BLACKBERRY && !UNITY_WINRT || UNITY_EDITOR
  hInput = Input.GetAxisRaw("Horizontal");
  myAnim.UpdateSpeed(hInput);
  if(Input.GetButtonDown("Jump"))
   Jump();
  #endif
 
  Move(hInput);
 }
 
 void Move(float horizonalInput)
 {
  if(!canMoveInAir && !isGrounded)
   return;
  
  Vector2 moveVel = myBody.velocity;
  moveVel.x = horizonalInput * speed;
  myBody.velocity = moveVel;
 }
 
 public void Jump()
 {
  if(isGrounded)
   myBody.velocity += jumpVelocity * Vector2.up;
 }
 
 public void StartMoving(float horizonalInput)
 {
  hInput = horizonalInput;
  myAnim.UpdateSpeed(horizonalInput);
 }
}






//**************//
//***Animator***//
//**************//

using UnityEngine;
using System.Collections;
 
public class AnimatorController : MonoBehaviour
{
 public static AnimatorController instance;
 
 Transform myTrans;
 Animator myAnim;
 Vector3 artScaleCache;
 
 void Start ()
 {
  myTrans = this.transform;
  myAnim = this.gameObject.GetComponent<Animator> ();
  instance = this;
 
  artScaleCache = myTrans.localScale;
 }
 
 void FlipArt(float currentSpeed)
 {
  if((currentSpeed < 0 && artScaleCache.x > 0)|| //going left AND faceing right OR...
   (currentSpeed > 0 && artScaleCache.x < 0)) //going right AND facing left
  {
   //flip the art
   artScaleCache.x *= -1;
   myTrans.localScale = artScaleCache;
  }
  
 }
 
 public void UpdateSpeed(float currentSpeed)
 {
  myAnim.SetFloat ("Speed", currentSpeed);
  FlipArt(currentSpeed);
 }
 
 public void UpdateIsGrounded(bool isGrounded)
 {
  myAnim.SetBool ("isGrounded", isGrounded);
 }
}
