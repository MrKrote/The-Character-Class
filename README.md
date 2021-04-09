# The-Character-Class

> Meta Keyword
> 
```
UPROPERTY(VisibleAnywhere,BlueprintReadOnly,Category = "Camera", meta = (AllowPrivateAcces = "true"))
```
***meta = (AllowPrivateAcces = "true") - > it will make accessible inside the blueprint that contains it but not outside of it.***

&nbsp;

> Character - Create Camera and SpringArm
> 
**.H**
```
UPROPERTY(VisibleAnywhere,BlueprintReadOnly,Category = "Camera", meta = (AllowPrivateAcces = "true"))
class USpringArmComponent* CameraBoom;
UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Camera", meta = (AllowPrivateAcces = "true"))
class UCameraComponent* FollowCamera;
```
**.CPP** 
**Constructor**
```
CameraBoom = CreateDefaultSubobject<USpringArmComponent>(TEXT("CameraBoom"));
CameraBoom->SetupAttachment(GetRootComponent());
CameraBoom->TargetArmLength = 600.0f; 
CameraBoom->bUsePawnControlRotation = true; 
FollowCamera = CreateDefaultSubobject<UCameraComponent>(TEXT("FollowCamera"));
FollowCamera->SetupAttachment(CameraBoom, USpringArmComponent::SocketName);
FollowCamera->bUsePawnControlRotation = false;
```
// Camera: (TIP) How to get 'over the shoulder' third person
```
CameraBoom->SocketOffset = FVector(0.f, 75.f, 50.f);
```
&nbsp;

> Character - MoveForward , MoveBackwards
> 

**.H**
```
UPROPERTY(VisibleAnywhere,BlueprintReadOnly,Category = "Camera")
float BaseTurnRate;
UPROPERTY(VisibleAnywhere,BlueprintReadOnly,Category = "Camera")
float BaseLookUpRate;

/** Called for forwards/backwards input */
void MoveForward(float Value);
/** Called for side to side input */
void MoveRight(float Value);
 
/** Called via input to turn at a given rate 
* @param Rate this is a normalized rate, i.e. 1.0 means 100% of desired turn rate
*/
void TurnAtRate(float Rate);
 
/** Called via input to look up/down at a  given rate
* @param Rate this is a normalized rate, i.e. 1.0 means 100% of desired up/down rate
*/
void LookUpAtRate(float Rate);
 
/** Getters */
FORCEINLINE class USpringArmComponent* GetCameraBoom() const { return CameraBoom; }
FORCEINLINE class UCameraComponent* GetFollowCamera() const { return FollowCamera; }
```
**.CPP** 
**Constructor**
```
BaseTurnRate = 65.0f;
BaseLookUpRate = 65.0f;
 ```
**MoveForward**
```
{
if ((Controller != nullptr) && (Value != 0.0f)){
const FRotator Rotation = Controller->GetControlRotation();
const FRotator YawRotation(0.0f, Rotation.Yaw, 0.0f);
const FVector Direction = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::X); 
AddMovementInput(Direction, Value);
}}
```
**MoveRight**
```
{
if ((Controller != nullptr) && (Value != 0.0f)){
const FRotator Rotation = Controller->GetControlRotation();
const FRotator YawRotation(0.0f, Rotation.Yaw, 0.0f);
const FVector Direction = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::Y); 
AddMovementInput(Direction, Value);
}}
```
**void AMain::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent) **
```
{
Super::SetupPlayerInputComponent(PlayerInputComponent);
check(PlayerInputComponent);
 
PlayerInputComponent->BindAxis("MoveForward", this, &AMain::MoveForward); // W and S key 
PlayerInputComponent->BindAxis("MoveRight", this, &AMain::MoveRight); // A and D key 
 
PlayerInputComponent->BindAxis("Turn", this, &APawn::AddControllerYawInput); // mouse turn
PlayerInputComponent->BindAxis("LookUp", this, &APawn::AddControllerPitchInput); // mouse look up down
PlayerInputComponent->BindAxis("TurnRate", this, &AMain::TurnAtRate);
PlayerInputComponent->BindAxis("LookUpRate", this, &AMain::LookUpAtRate);
 
PlayerInputComponent->BindAction("Jump", IE_Pressed, this, &ACharacter::Jump);
PlayerInputComponent->BindAction("Jump", IE_Released, this, &ACharacter::StopJumping);
}
```
**void AMain::TurnAtRate(float Rate)**
{
AddControllerYawInput(Rate * BaseTurnRate * GetWorld()->GetDeltaSeconds());
}
 ```
 ```
**void AMain::LookUpAtRate(float Rate)**
{
AddControllerPitchInput(Rate * BaseLookUpRate * GetWorld()->GetDeltaSeconds());
}
```
> Character - CUSTOMIZING - Don't rotate when the camera rotates
> 

```/*
Don't rotate when the controller rotates
* Let that just affect the camera
*/
bUseControllerRotationYaw = false;
bUseControllerRotationPitch = false;
bUseControllerRotationRoll = false;
//Configure character movement */
GetCharacterMovement()->bOrientRotationToMovement = true; // Character moves in the direction of input...
GetCharacterMovement()->RotationRate = FRotator(0.0f, 440.0f, 0.0f); // ... at this rotation rate ( rotate faster )
GetCharacterMovement()->JumpZVelocity = 450.0f; // high of jump
GetCharacterMovement()->AirControl = 0.2f;   // allow to move the character in space while it is in the air
/** Set Size for collision capsule */
GetCapsuleComponent()->SetCapsuleSize(48.0f, 105.0f);
```

&nbsp;

> Animation
> **1. Create an AnimInstance C++ Class**
**.H**
```
public:
virtual void NativeInitializeAnimation() override;
 
UFUNCTION(BlueprintCallable, Category = "AnimationProperties")
void UpdateAnimationProperties();
 
UPROPERTY(EditAnywhere,BlueprintReadOnly,Category = "Movement")
float MovementSpeed;
	
UPROPERTY(EditAnywhere,BlueprintReadOnly,Category = "Movement")
bool bIsInAir;
 
UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Movement")
class APawn* Pawn;
```
**.CPP**
```
NativeInitializeAnimation()
{
if (Pawn == nullptr)
{
Pawn = TryGetPawnOwner();
}
}

UpdateAnimationProperties()
{
if (Pawn == nullptr)
{
Pawn = TryGetPawnOwner();
}
if(Pawn)
{
FVector Speed = Pawn->GetVelocity();
FVector LateralSpeed = FVector(Speed.X, Speed.Y, 0.0f);
MovementSpeed = LateralSpeed.Size();
bIsInAir = Pawn->GetMovementComponent()->IsFalling();
}
}
```

**#2 Animation*+
**2.** Create an Animation -> Animation Blend Space 1D -> Drag and Drop the Anims ( press shift u can see how is it )

Axis Settings -> Horizontal Axis -> Give a Name ( like Speed ) + Maximum Axis Value ( like 375 )

**3.** Create an Animation -> Animation Blueprint. Than File -> Reparent BP -> Choise C++ class.So we can access in Blueprint to our custom AnimationProperties function. ( like Tick () method )

**4.** AnimGraph -> Right Click , New state machine (Name = Locomotion ) -> Clink in and Add State ( Name = Idle / Walk / Run ) -> Now we can drag and drop our BlendSpace what we created in 2. and the movement speed.

**5.** Main_BP -> Animation -> Animation Mode should be Use Animation Blueprint and AnimClass the Animation Class

**6.**In Locomotion from Entry -> Idle/Walk/Run (add state)

From Idle/Walk/Run -> JumpStart (add state) -> jumping up -> transition rule -> is in Air

From JumpStart -> InAir (add state) -> falling_idle -> transition rule -> current time (ratio) jumping up -> >=

From InAir -> Jump End ( add state ) -> jumping_down -> transition rule -> IsInAir -> NOT BOOL 

From JumpEnd -> Idle/Walk/Run -> transition rule -> TimeRemaining (ratio) -> <

**7.** bind in MainAnim_BP the Event Blueprint Update Animation with Update Animation Properties






