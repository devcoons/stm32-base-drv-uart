# UART DRIVER

UART driver reusable on different hardwares.

## Supported Hardware

- [supported]
- [supported]
- [supported]

### Functions Guide

- `uart_initialize` : initialize the interfaces.
- `uart_send` : sends data as raw bites.
- `uart_send_message` : sends data as message.
- `uart_send_lowpulse` : sends a low pulse.
- `uart_callback_add` : adds a callback.

## How to use

- Include the header file `drv_uart.h`.
- Enable the UART peripheral in the `.ioc` file.
- Create an instance of `uart_t` to be used as the handler and afterwards create an array of `uart_interface_t` (uart_interface_t interfaces[]={interface1, interface2...}). For each interface, define:
	- `interface.mx_init`: UART initializing function generated by the configuration tool.
	- `interface.huart` : handler of choice (ex. huart4).
	- `interface.callbacks` :array of `uart_callback_t` instances:
		- .void (* callback)(uint8_t *buffer, uint32_t size) : pointer to the callback function.
		- An example of callback function:
		  ```C
			static void cb_uart(uint8_t *buffer, uint32_t size)
			{
				string message_base64;
				uint8_t ser_dt[13];
			
				base64_decode(buffer, size, ser_dt);
					
				iqueue_enqueue(&q_uart, &dec_frame);
			} 
		 
	- `interface. tx_port` : port used for the low pulse (define only if needed).
	- `interface. tx_pin` : pin used for the low pulse (define only if needed).
	- `interface. in_buffer` : address of 8-bit array for reception/transmission buffer.
	- `interface. in_buffer_sz` : position in the buffer.
	- `interface. in_buffer_tsz` : buffer size.
	- `interface.raw_timout ` : we assume the message ends after the timeout.
	- `interface.max_raw_timout ` : maximum timeout for a raw message
	
- Use the ```uart_send_message``` function to send a message:
	```C
	uart_send_message(&uart_instance, message, b64_sz);
	```
- Messages are read thought the callback function. To add a callback function use ```uart_callback_add```:
	```C
	uart_callback_add(&uart_instance, cb_uart);
	```


## Example

Let's consider a NUCLEO-H7A3ZI-Q. UART4 has been enabled in the .ioc file

```C

static uint8_t uart_buffer[256];

uart_t uart_instance_ = {
		.mx_init = MX_UART4_Init,
		.in_buffer_tsz = 250,
		.in_buffer_sz = 0,
		.huart = &huart4,
		.in_buffer = uart_buffer,
};

uart_initialize(&uart_instance);

uart_send_message(&uart_instance, message, b64_sz);

uint8_t message[32];

uart_callback_add(&uart_instance, cb_uart);



```
