---
{"dg-publish":true,"permalink":"/0-notes/eth-embedded-system/","noteIcon":"","created":"2024-01-27T07:52:10.084+01:00","updated":"2024-01-27T07:55:56.470+01:00"}
---

## Lecture 1

{% pdf "/assets/pdf/eth_es_l1.pdf" %}
{% pdf "/assets/pdf/eth_es_lab0.pdf" %}

### Takeaway
#### Task1
- The initial states of registers are unknown, therefore, it is always essential to first initialize the states of all registers to be used.
        - Given by the TA

> Setting one LED with GPIO_setOutputHighOnPin and one with GPIO_setOutputLowOnPin ensure the two LEDs have different starting states. The LEDs may also blink alternating without this initialisation, but only if you are lucky with the initial state of the registers. Therefore it is important to initialise all the registers properly to ensure deterministic behaviour of your program, as embedded systems often do not intialise all their registers to a defined state at startup.

#### Task 2
- What is *floating pin*?
    - Ref: https://www.mouser.com/blog/dont-leave-your-pins-floating

> The voltage of the pin is underterminate, which is floating (some kind of electrical noise). Usually, the floating pin is not connected either to the vcc or the ground. Best practice is to add pull-up registers or pull-down registers. In some cases, both of them can be used. But in certain case, one of them has to be used. For example, in I2C, the pull-up reigster must be used (configure something???)

- How to mitigate the influence of polling on the blinking frequency?

> Polling consumes more time in the for loop. Considering the empty for loop, where iteration takes time. Extra consumed time can be calculated to be equvilant to the iteration time. By approximately modifying the counter, the blinking frequency can be changed.

#### Task 3

- `UART` configuration shorthand notation

> 8N1 means
8 bits per packet
No parity bits
1 stop bit

- UART configuration

> low frequency or the oversampling baud rate generation.
The low frequency baud rate generation minimizes the energy consumption of the chip, while being a bit more error prone. Therefore, in case the oversampling baud rate generation is available and the energy consumption is not crucial, the oversampling baud should be chosen.

- [TODO] Implement another method of using interrupt for UART
### Solution
code is lost after re-installing windows 10 system...

---
## Lecture 2

{% pdf "/assets/pdf/eth_es_l3.pdf" %}

{% pdf "/assets/pdf/eth_es_lab2.pdf" %}

- [TODO] Append comparison between polling and interrupt (focused on timing)

### Take-away
- mechanisms of initiating actions
    - event-triggered
    - time-triggered
- polling and interrupt
    - polling
        - disadvantage
            - keeping system in the active state consumes an over-proportional amount of energy
        - advantage
            - quick reaction time
        - when using polling
            - deterministic arrival time of events (e.x. the button is pressed every second)
            - *a condition has to be checked with a high frequency*
    - interrupt
        - advantage
            - system can stay in low power sleep state before interrupt
        - disadvantage
            - transition from the sleep to active state requires time which introduces a delay
            - reaction might not be instant due to block from interrupts with higher priority
        - when using interrupt
            - asynchronous or infrequent events
- system states
    - power mode level
- interrupt flow
{% include figure.html path="assets/img/eth_es/eth_es_interrupt.png" class="img-fluid rounded z-depth-1" %}

- **Highlights**
    - interrupt flag register (IFG)
    - interrupt enable (IE) (peripheral + internal controller)
    - global interrupt enable (GIE)
        - non-maskable interrupts (NMI)
    - inside the ISR
        - identify the interrupt’s exact source
        - mask other interrupts
        - set flags for MCU (indication of task response to the interrupt)
- GPIO pin configuration
    - pull up register
        - deterministic status of the pin
        - default status is low level
- ISR (interrupt service routine)
    - only allowed to modify global variables
    - no arguments or return values
- library (example) code
    ```c
    (void (*)(void))((uint32_t)&__STACK_END)
    // __STACK_END is a linker variable
    // cast the stack pointer to a function pointer?
    ```

### Solution
#### Task 1
```c
#include "lab2.h"                              // Lab specific defines/declarations

/* Global variables */
volatile bool button1Flag;                     // Flag to indicate that button S1 has been pressed
volatile bool button2Flag;                     // Flag to indicate that button S2 has been pressed
volatile uint32_t count1 = 0;                  // Variable to count interrupts produced by S1
volatile uint32_t count2 = 0;                  // Variable to count interrupts produced by S2

void task1(void)
{
    /* Setup UART */
    uart_init(UART_BAUDRATE);

    /* Configuring P2.1 (LED2 green) and P2.2 (LED2 blue) as output */
    ////////////////////////////////////////////////////////////////////////////
    GPIO_setAsOutputPin(GPIO_PORT_P2, GPIO_PIN1 | GPIO_PIN2);

    /* Configuring P1.1 (Button S1) and P1.4 (Button S2) as an input */
    ////////////////////////////////////////////////////////////////////////////
    GPIO_setAsInputPinWithPullUpResistor(GPIO_PORT_P1, GPIO_PIN1 | GPIO_PIN4);


    /* Configure interrupts for buttons S1 and S2 */
    ////////////////////////////////////////////////////////////////////////////
    /*Interrupt should be triggered when the button is released -> from low to high*/
    GPIO_interruptEdgeSelect(GPIO_PORT_P1, GPIO_PIN1 | GPIO_PIN4, GPIO_LOW_TO_HIGH_TRANSITION);
    GPIO_clearInterruptFlag(GPIO_PORT_P1, GPIO_PIN1 | GPIO_PIN4);

    /* Enable interrupts for both GPIO pins */
    /* Enable interrupts on Port 1 */
    /* Enable interrupts globally */
    ////////////////////////////////////////////////////////////////////////////
    /*Enable interrupts for certain GPIO pins*/
    GPIO_enableInterrupt(GPIO_PORT_P1, GPIO_PIN1 | GPIO_PIN4);
    /*Enable interrupts for certain Port*/
    Interrupt_enableInterrupt(INT_PORT1);
    /*Enable global interrupts*/
    Interrupt_enableMaster();

    /* Initially turn LEDs off */
    ////////////////////////////////////////////////////////////////////////////
    GPIO_setOutputLowOnPin(GPIO_PORT_P2, GPIO_PIN1 | GPIO_PIN2);


    /* Code that is executed when processor is not busy with handling an interrupt */
    while (1)
    {
        // Sleep while waiting for an interrupt
        PCM_gotoLPM3();
        // After interrupt occurred
        if (button1Flag)
        {
            /* Increase and send counter for button S1 via UART */
            count1 += 1;
            uart_println("S1: %d", count1);
            button1Flag = false;
        }
        if (button2Flag)
        {
            /* Increase and send counter for button S2 via UART */
            count2 += 1;
            uart_println("S2: %d", count2);
            button2Flag = false;
        }
    }
}

/*
* PORT 1 interrupt service routine
*/
void PORT1_IRQHandler(void)
{
    uint32_t status;

    /* Get the content of the interrupt status register of port 1 */
    status = GPIO_getEnabledInterruptStatus(GPIO_PORT_P1);

    if (status & GPIO_PIN1) {
        GPIO_toggleOutputOnPin(GPIO_PORT_P2, GPIO_PIN1);
        button1Flag = true;
        GPIO_clearInterruptFlag(GPIO_PORT_P1, GPIO_PIN1);
    } else if (status & GPIO_PIN4) {
        GPIO_toggleOutputOnPin(GPIO_PORT_P2, GPIO_PIN2);
        button2Flag = true;
        GPIO_clearInterruptFlag(GPIO_PORT_P1, GPIO_PIN4);
    }
}
```

### Task 2
```c

#include "lab2.h"                       // Lab specific defines/declarations

/* Application defines  */
#define TA0_CCR0_BREATH           499
#define PWM_CCR0                  127
#define COMPARE_VAL_INC_STEP        1
#define TA0_CCR1_INIT_VAL         126

/* Port mapper configuration register */
const uint8_t port_mapping[] =
{
    PMAP_TA1CCR1A, PMAP_NONE, PMAP_NONE, PMAP_NONE, PMAP_NONE, PMAP_NONE, PMAP_NONE, PMAP_NONE
};

void task2(void)
{
    /* Timer_A0 up mode configuration parameters */
    const Timer_A_UpModeConfig upConfigA0 =
    {
        TIMER_A_CLOCKSOURCE_SMCLK,     // SMCLK clock source (3MHz)
        TIMER_A_CLOCKSOURCE_DIVIDER_64,     // SMCLK/??
        TA0_CCR0_BREATH,     // Value in CCR0; NOTE: limited to 16 bit!
        TIMER_A_TAIE_INTERRUPT_DISABLE,     // Disable timer roll-over interrupt
        TIMER_A_CCIE_CCR0_INTERRUPT_ENABLE,     // Enable CCR0 interrupt
        TIMER_A_DO_CLEAR,      // Clear value in the timer counter register
    };

    /* Configuring P1.0 (LED1) as output */
    ////////////////////////////////////////////////////////////////////////////
    GPIO_setAsOutputPin(GPIO_PORT_P1, GPIO_PIN0);
    /*Turn off the LED1*/
    GPIO_setOutputLowOnPin(GPIO_PORT_P1, GPIO_PIN0);

    /* Configuring Timer_A0 for Up mode */
    ////////////////////////////////////////////////////////////////////////////
    Timer_A_configureUpMode(TIMER_A0_BASE, &upConfigA0);

    /* Clear interrupt */
    ////////////////////////////////////////////////////////////////////////////
    Timer_A_clearCaptureCompareInterrupt(TIMER_A0_BASE, TIMER_A_CAPTURECOMPARE_REGISTER_0);
    Interrupt_enableInterrupt(INT_TA0_0);
    Interrupt_enableMaster();
    Timer_A_startCounter(TIMER_A0_BASE, TIMER_A_UP_MODE);

    /* Remapping TA1CCR1 output pin to P2.0 (red LED of LED2) */
    PMAP_configurePorts((const uint8_t *) port_mapping,
                        PMAP_P2MAP,
                        1,
                        PMAP_DISABLE_RECONFIGURATION);

    /* Set pin P2.0 as output pin */
    GPIO_setAsPeripheralModuleFunctionOutputPin(GPIO_PORT_P2,
                                                GPIO_PIN0,
                                                GPIO_PRIMARY_MODULE_FUNCTION);

    /* Timer_A1 up mode configuration parameters */
    const Timer_A_UpModeConfig upConfigA1 =
    {
        TIMER_A_CLOCKSOURCE_SMCLK,     // SMCLK clock source
        TIMER_A_CLOCKSOURCE_DIVIDER_1,     // SMCLK/1 = 3MHz
        PWM_CCR0,     // Value in CCR0; NOTE: limited to 16 bit!
        TIMER_A_TAIE_INTERRUPT_DISABLE,     // Disable Timer interrupt
        TIMER_A_CCIE_CCR0_INTERRUPT_DISABLE,     // Disable CCR0 interrupt
        TIMER_A_DO_CLEAR,      // Clear value
    };

    /* Timer_A1 compare configuration parameters */
    Timer_A_CompareModeConfig compareConfigA1 =
    {
        TIMER_A_CAPTURECOMPARE_REGISTER_1,     // Use CCR1 as compare register
        TIMER_A_CAPTURECOMPARE_INTERRUPT_DISABLE,     // Disable CCR interrupt
        TIMER_A_OUTPUTMODE_TOGGLE_RESET,     // Toggle-reset output mode
        TA0_CCR1_INIT_VAL,      // Compare value (CCR1)
    };

    ////////////////////////////////////////////////////////////////////////////
    Timer_A_initCompare(TIMER_A1_BASE, &compareConfigA1);
    Timer_A_configureUpMode(TIMER_A1_BASE, &upConfigA1);
    Timer_A_startCounter(TIMER_A1_BASE, TIMER_A_UP_MODE);

    /* Sleeping when not in use */
    while (1)
    {
        PCM_gotoLPM3();
    }
}

void TA0_0_IRQHandler(void)
{
    /* Toggle LED1 */
    ////////////////////////////////////////////////////////////////////////////
    GPIO_toggleOutputOnPin(GPIO_PORT_P1, GPIO_PIN0);

    /* Definitions for Task 2.3 */
    static int16_t pwmCompareVal = PWM_CCR0;    // Current duty cycle value
    static bool goUp = false;                   // Flag to indicate whether compare
                                                // value is currently increased or decreased

    /* Update compare value variable */
    ////////////////////////////////////////////////////////////////////////////
    if (goUp)
    {
        pwmCompareVal += COMPARE_VAL_INC_STEP;
    } else
    {
        pwmCompareVal -= COMPARE_VAL_INC_STEP;
    }

    /* Invert the direction of increase/decrease of the duty cycle */
    ////////////////////////////////////////////////////////////////////////////
    if (pwmCompareVal == 0)
    {
        goUp = true;
    } else if (pwmCompareVal == PWM_CCR0)
    {
        goUp = false;
    }

    /* Update the compare value of Timer A1 (linear increase/decrease) */
    uint16_t pwmCompareValCurr = pwmCompareVal;
    Timer_A_setCompareValue(TIMER_A1_BASE,
                            TIMER_A_CAPTURECOMPARE_REGISTER_1,
                            pwmCompareValCurr);

    /* Clear interrupt of timer A0 */
    ////////////////////////////////////////////////////////////////////////////
    Timer_A_clearCaptureCompareInterrupt(TIMER_A0_BASE, TIMER_A_CAPTURECOMPARE_REGISTER_0);
}
```