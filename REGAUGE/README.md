# *REGAUGE*
## Game Summary
Hop in a mech and when yours explodes get to cover and run away from the other players that are trying to stomp you, until you get your mech drop and *REGAUGE*.

## Player
 Implementing couch co-op with Unity's input system required some structure for REGAUGE since the player needed to swap between multiple different kinds of characters. The implementation I went with is having a controller that can attach characters to itself and most of the time handle the logic for switching characters.

  <Details>
 <summary> Code </summary>

```cs
using UnityEngine;
using UnityEngine.InputSystem;

public class PlayerCharacterBase : MonoBehaviour
{
    public GunScript gun;
    public SetRotationTo aimer;
    [HideInInspector] public PlayerController playerController;
    [HideInInspector] public PlayerMovement playerMovement;
    [HideInInspector] public Mouse mouse;
    
    public float deadzone = 0.25f;

    public bool fireDown { get; private set; }
    bool dashDown;
    bool ActionDown;

    public virtual void Awake()
    {
        playerMovement = GetComponent<PlayerMovement>();
        playerMovement.playerCharacter = this;
        deadzone = PlayerPrefs.GetFloat(Settings.deadZoneKey);
    }

    public virtual void Start() { }

    public void FireInternalPressed(InputAction.CallbackContext context)
    {
        fireDown = true;
        FirePressed();
    }
    public void FireInternalReleased(InputAction.CallbackContext context)
    {
        fireDown = false;
        FireReleased();
    }

    public virtual void FirePressed() { }
    public virtual void FireHeld() { }
    public virtual void FireReleased() { }


    public void DashInternalPressed(InputAction.CallbackContext context)
    {
        dashDown = true;
        DashPressed();
    }
    public void DashInternalReleased(InputAction.CallbackContext context)
    {
        dashDown = false;
        DashReleased();
    }

    public virtual void DashPressed() { }
    public virtual void DashHeld() { }
    public virtual void DashReleased() { }

    public virtual void SouthPressed(InputAction.CallbackContext context) { }
    public virtual void SouthReleased(InputAction.CallbackContext context) { }

    public virtual void EastPressed(InputAction.CallbackContext context) { }
    public virtual void EastReleased(InputAction.CallbackContext context) { }

    public virtual void NorthPressed(InputAction.CallbackContext context) { }
    public virtual void NorthReleased(InputAction.CallbackContext context) { }

    public virtual void WestPressed(InputAction.CallbackContext context) { }
    public virtual void WestReleased(InputAction.CallbackContext context) { }


    public virtual void Move(Vector2 input, Vector2 aim, Vector2 mouse) { playerMovement.Move(input, aim, mouse); }
    public virtual void Rotate(Vector2 input, Vector2 input2, Vector2 mouse) { }

    public virtual void OnDisconnect()
    {
        playerController = null;
        Destroy(playerMovement);
    }

    public void Update()
    {
        if (fireDown) { FireHeld(); }
        if (dashDown) { DashHeld(); }
    }

    private void OnDisable()
    {
        if (playerController && playerController.playerCharacter == this) { playerController.RemovePlayerCharacter(); }
    }

    public virtual void DestroyCharacter() { }
}

```

 </Details>

 ### Rotation
The mech rotates a lot and this requires a bit of handling since there are also a few different rotation ways.

 <img width="557" height="486" alt="Player Rotation" src="https://github.com/user-attachments/assets/5f0c118a-b8a4-4c9c-b874-cf5a3905d30a" />
The rotations that are being tracked by the player are two input vectors for movement and aiming, then there is the current direction of the legs and the torso which is also being smoothed. Then there is also a dash state where the player slowly becomes more responsive, the first thing that unlocks for the player after the dash is the movement rather than the ability to shoot in the aim direction. Lastly the torso starts to rotate in the input direction again. The movement direction is also used instead of the aim direction when aim input is given, the reason for this was to make the game more accessible and give the player a simpler way to interact with the game.


 <Details>
 <summary> Code </summary>

```cs
using System.Collections;
using UnityEngine;

public class PlayerTorsoRotation : MonoBehaviour
{
    [HideInInspector] public PlayerCharacterBase playerCharacter;
    [HideInInspector] public PlayerMovement playerMovement;
    Animator animator;

    [SerializeField, Tooltip("x = 0 when trying to look backwards. x = 1 when trying to look forward")] AnimationCurve turnCurve;
    [SerializeField] float turnSpeed;
    [SerializeField] float timeForRotationVisuals;
    [SerializeField] AnimationClip rotationAnim;

    [HideInInspector] public Vector3 aimDirection = Vector3.back;
    Vector3 currentLookDirection = Vector3.back;

    private void Awake()
    {
        animator = GetComponentInChildren<Animator>();
        aimDirection = Vector3.back;
    }

    public void Startup(float time = 0)
    {
        StopAllCoroutines();
        StartCoroutine(StaggeredRotation(time));
    }

    public void SnapRotation() { currentLookDirection = aimDirection; }

    public void Rotate(Vector2 input, Vector2 input2, Vector2 mouse)
    {
        Vector2 target = input;
        if (input == Vector2.zero)
        {
            if (playerCharacter.mouse != null) { target = mouse; }
            else { target = input2; }
        }
        if (target != Vector2.zero && target.sqrMagnitude > Mathf.Pow(playerCharacter.deadzone, 2))
        {
            aimDirection = new Vector3(target.x, 0, target.y);

            float dotProduct = Vector3.Dot(new Vector3(target.x, 0, target.y), new Vector3(currentLookDirection.x, 0, currentLookDirection.y));
            dotProduct = (dotProduct + 1) / 2;
            currentLookDirection = Vector3.Slerp(currentLookDirection, new Vector3(target.x, 0, target.y), turnCurve.Evaluate(dotProduct) * turnSpeed * Time.deltaTime);
        }
    }

    IEnumerator StaggeredRotation(float time)
    {
        yield return new WaitForSeconds(time);
        while (true)
        {
            SetRotation();
            yield return new WaitForEndOfFrame();
        }
    }

    public void SetRotation()
    {
        Vector3 relativeLookDirection;
        if (playerMovement.inOverrideAnim)
        {
            relativeLookDirection = Vector3.forward;
        }
        else
        {
            relativeLookDirection = playerMovement.meshRoot.InverseTransformVector(currentLookDirection);
        }
        Debug.DrawLine(transform.position + Vector3.up * 2, transform.position + relativeLookDirection + Vector3.up * 2, Color.magenta, Time.deltaTime);

        animator.Play(rotationAnim.name, animator.GetLayerIndex("RotateBody"), ((Mathf.Atan2(relativeLookDirection.x, -relativeLookDirection.z) / Mathf.PI) + 1) / 2);
    }
}

```

 </Details>

 ## Tiniest Map
 The game had a bit of a problem during a playtest which was that it was really boring to keep playing for a while. This was known ofcourse however the focus until then was to make a great base. So when we felt that we could spend some time we started adding to the breadth of the game including new maps.


The design behind this behemoth of a map was to be a palette cleansers between the more intricate and longer lasting maps that already existed. I wanted it to be a bunch of repeated 1v1s since all the other maps consisted of mostly large open areas where one can not completely have an uninterrupted fight.



