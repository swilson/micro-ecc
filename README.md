micro-ecc
==========

A small ECDH and ECDSA implementation for 32-bit microcontrollers.

Features
--------

 * Written in C, with optional inline assembly for ARM and Thumb platforms.
 * Small code size: as low as 1800 bytes when compiled for Thumb (eg, Cortex-M0).
 * Reasonably fast: on an LPC1114 at 48MHz (ARM Cortex-M0, 32-cycle 32x32 bit multiply), 192-bit ECDH shared secret calculation takes as little as ~150ms (depending on selected optimizations).
 * Support for 4 standard curves: secp128r1, secp192r1, secp256r1, and secp384r1
 * Optional optimizations so you can choose between code size and speed.

Usage Notes
-----------

**NOTE:** This implementation is not designed to be resistant to timing attacks or other side-channel attacks. To thwart timing attacks, you can use a timer to ensure that all ECDH operations take the same amount of time (ie, the longest time it would ever take). You can determine the maximum time for ECDH by using a private key that is all 0xffffffff.

#### Integer Representation ####

To reduce code size, all large integers are represented using little-endian words - so the least significant word is first. For example, the standard representation of the prime modulus for the curve secp128r1 is `FFFFFFFD FFFFFFFF FFFFFFFF FFFFFFFF`; in micro-ecc, this would be represented as `uint32_t p[4] = {0xffffffff, 0xffffffff, 0xffffffff, 0xfffffffd};`.

#### Generating Keys ####

You can use the `makekeys` program in the `apps` directory to generate keys (on Linux or OS X). You can run `make` in that directory to build for your native platform (or use [emk](http://kmackay.ca/emk)). To generate a single public/private key pair, run `makekeys`. It will print out the public and private keys in a representation suitable to be copied into your source code. You can generate multiple key pairs at once using `makekeys <n>` to generate n keys.

#### Using the Code ####

I recommend just copying (or symlink) ecc.h and ecc.c into your project.

Speed and Size
--------------

Available optimizations are:
 * `ECC_SQUARE_FUNC` - Use a separate function for squaring.
 * `ECC_USE_NAF` - Use a non-adjacent form for scalar representation when doing point multiplication.
 * `ECC_ASM` - Choose the type of inline assembly to use. The available options are `ecc_asm_none`, `ecc_asm_thumb`, `ecc_asm_thumb2`, and `ecc_asm_arm`.

All tests were performed on an LPC1114 running at 48MHz. The listed code sizes include all code and data required by the micro-ecc library (including `aebi_lmul` when not using assembly),
but do not include the sizes of the code using the library functions.

The following compiler options were used (using gcc 4.8):
 * Compile: `-mcpu=cortex-m0 -mthumb -ffunction-sections -fdata-sections -Os`
 * Link: `-mcpu=cortex-m0 -mthumb -nostartfiles -nostdlib -Wl,--gc-sections`

### Effect of optimization settings ###

These tests were performed using the curve secp192r1. Only ECDH code was used (no ECDSA).

#### ECC_ASM defined to ecc_asm_none ####

<table>
	<tr>
		<th>Optimizations:</th>
		<th>none</th>
		<th>ECC_SQUARE_FUNC</th>
		<th>ECC_USE_NAF</th>
		<th>both</th>
	</tr>
	<tr>
		<td><em>ECDH time (ms):</em></td>
		<td>431.9</td>
		<td>393.0</td>
		<td>369.5</td>
		<td>337.2</td>
	</tr>
	<tr>
		<td><em>Code size (bytes):</em></td>
		<td>1948</td>
		<td>2152</td>
		<td>2208</td>
		<td>2412</td>
	</tr>
</table>

#### ECC_ASM defined to ecc_asm_thumb ####

<table>
	<tr>
		<th>Optimizations:</th>
		<th>none</th>
		<th>ECC_SQUARE_FUNC</th>
		<th>ECC_USE_NAF</th>
		<th>both</th>
	</tr>
	<tr>
		<td><em>ECDH time (ms):</em></td>
		<td>183.9</td>
		<td>172.6</td>
		<td>158.2</td>
		<td>147.7</td>
	</tr>
	<tr>
		<td><em>Code size (bytes):</em></td>
		<td>1772</td>
		<td>1920</td>
		<td>2032</td>
		<td>2180</td>
	</tr>
</table>

### ECDH for different curves ###

In these tests, `ECC_ASM` was defined to `ecc_asm_thumb` in all cases.

#### No other optimizations (smallest code size) ####

<table>
	<tr>
		<th>Curve:</th>
		<th>secp128r1</th>
		<th>secp192r1</th>
		<th>secp256r1</th>
		<th>secp384r1</th>
	</tr>
	<tr>
		<td><em>ECDH time (ms):</em></td>
		<td>85.5</td>
		<td>183.9</td>
		<td>473.6</td>
		<td>1405.8</td>
	</tr>
	<tr>
		<td><em>Code size (bytes):</em></td>
		<td>1908</td>
		<td>1772</td>
		<td>2104</td>
		<td>1832</td>
	</tr>
</table>

#### All optimizations (fastest) ####

<table>
	<tr>
		<th>Curve:</th>
		<th>secp128r1</th>
		<th>secp192r1</th>
		<th>secp256r1</th>
		<th>secp384r1</th>
	</tr>
	<tr>
		<td><em>ECDH time (ms):</em></td>
		<td>75.9</td>
		<td>147.7</td>
		<td>385.8</td>
		<td>1128.2</td>
	</tr>
	<tr>
		<td><em>Code size (bytes):</em></td>
		<td>2332</td>
		<td>2180</td>
		<td>2516</td>
		<td>2244</td>
	</tr>
</table>

### ECDSA speed and combined code size ###

In these tests, the measured speed is the time to verify an ECDSA signature. The measured code size is the combined code size for ECDH and ECDSA.

TODO

