using UnityEngine;

public class RealisticCarController : MonoBehaviour
{
    [Header("Real Drive: Open City")]

    [Header("Wheel Colliders")]
    public WheelCollider frontLeftCollider;
    public WheelCollider frontRightCollider;
    public WheelCollider rearLeftCollider;
    public WheelCollider rearRightCollider;

    [Header("Wheel Models")]
    public Transform frontLeftWheel;
    public Transform frontRightWheel;
    public Transform rearLeftWheel;
    public Transform rearRightWheel;

    [Header("Car Settings")]
    public float motorPower = 1800f;
    public float brakePower = 3500f;
    public float handbrakePower = 7000f;
    public float maxSteeringAngle = 32f;
    public float maxSpeed = 180f;

    [Header("Stability")]
    public float downforce = 80f;
    public Vector3 centerOfMass = new Vector3(0f, -0.45f, 0f);

    private Rigidbody rb;

    private float steeringInput;
    private float throttleInput;
    private float brakeInput;
    private bool handbrake;

    public float CurrentSpeed
    {
        get
        {
            if (rb == null) return 0f;
            return rb.linearVelocity.magnitude * 3.6f;
        }
    }

    void Start()
    {
        rb = GetComponent<Rigidbody>();
        rb.centerOfMass = centerOfMass;
    }

    void Update()
    {
        // PC controls for testing
        steeringInput = Input.GetAxis("Horizontal");

        float vertical = Input.GetAxis("Vertical");

        throttleInput = Mathf.Clamp01(vertical);
        brakeInput = Mathf.Clamp01(-vertical);

        handbrake = Input.GetKey(KeyCode.Space);

        UpdateWheelVisuals();
    }

    void FixedUpdate()
    {
        Steering();
        Drive();
        Brake();
        ApplyDownforce();
    }

    void Steering()
    {
        float steering = steeringInput * maxSteeringAngle;

        frontLeftCollider.steerAngle = steering;
        frontRightCollider.steerAngle = steering;
    }

    void Drive()
    {
        float power = 0f;

        if (CurrentSpeed < maxSpeed)
        {
            power = throttleInput * motorPower;
        }

        rearLeftCollider.motorTorque = power;
        rearRightCollider.motorTorque = power;
    }

    void Brake()
    {
        float normalBrake = brakeInput * brakePower;

        frontLeftCollider.brakeTorque = normalBrake;
        frontRightCollider.brakeTorque = normalBrake;
        rearLeftCollider.brakeTorque = normalBrake;
        rearRightCollider.brakeTorque = normalBrake;

        if (handbrake)
        {
            rearLeftCollider.brakeTorque = handbrakePower;
            rearRightCollider.brakeTorque = handbrakePower;
        }
    }

    void ApplyDownforce()
    {
        rb.AddForce(-transform.up * downforce * rb.linearVelocity.magnitude);
    }

    void UpdateWheelVisuals()
    {
        UpdateWheel(frontLeftCollider, frontLeftWheel);
        UpdateWheel(frontRightCollider, frontRightWheel);
        UpdateWheel(rearLeftCollider, rearLeftWheel);
        UpdateWheel(rearRightCollider, rearRightWheel);
    }

    void UpdateWheel(WheelCollider collider, Transform wheel)
    {
        if (collider == null || wheel == null)
            return;

        collider.GetWorldPose(out Vector3 position, out Quaternion rotation);

        wheel.position = position;
        wheel.rotation = rotation;
    }

    // Mobile control functions

    public void SetSteering(float value)
    {
        steeringInput = Mathf.Clamp(value, -1f, 1f);
    }

    public void SetThrottle(float value)
    {
        throttleInput = Mathf.Clamp01(value);
    }

    public void SetBrake(float value)
    {
        brakeInput = Mathf.Clamp01(value);
    }

    public void SetHandbrake(bool value)
    {
        handbrake = value;
    }
}
