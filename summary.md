# Spectre Attacks: Exploiting Speculative Execution SUMMARY



Computations performed by physical devices often leave
observable side effects beyond the computation’s nominal
outputs. Side-channel attacks focus on exploiting these side
effects to extract otherwise-unavailable secret information.
Since their introduction in the late 90’s [43], many physical
effects such as power consumption [41, 42], electromagnetic
radiation [58], or acoustic noise [20] have been leveraged to
extract cryptographic keys as well as other secrets.
Physical side-channel attacks can also be used to extract
secret information from complex devices such as PCs and

some attacks exploit software vulnerabilities (such as buffer
overflows [5] or double-free errors [12]), other software attacks
leverage hardware vulnerabilities to leak sensitive information.
Attacks of the latter type include microarchitectural attacks
exploiting cache timing [8, 30, 48, 52, 55, 69, 74], branch
prediction history [1, 2], branch target buffers [14, 44] or open
DRAM rows [56]. Software-based techniques have also been
used to mount fault attacks that alter physical memory [39] or
internal CPU values [65].
Several microarchitectural design techniques have facilitated
the increase in processor speed over the past decades. One such
advancement is speculative execution, which is widely used
to increase performance and involves having the CPU guess
likely future execution directions and prematurely execute
instructions on these paths. More specifically, consider an
example where the program’s control flow depends on an
uncached value located in external physical memory. As this
memory is much slower than the CPU, it often takes several
hundred clock cycles before the value becomes known. Rather
than wasting these cycles by idling, the CPU attempts to guess
the direction of control flow, saves a checkpoint of its register
state, and proceeds to speculatively execute the program on the
guessed path. When the value eventually arrives from memory,
the CPU checks the correctness of its initial guess. If the
guess was wrong, the CPU discards the incorrect speculative
execution by reverting the register state back to the stored
checkpoint, resulting in performance comparable to idling.
However, if the guess was correct, the speculative execution
results are committed, yielding a significant performance gain.

We present a class of microarchitectural attacks which we call Spectre attacks. At a high
level, Spectre attacks trick the processor into speculatively
executing instruction sequences that should not have been
executed under correct program execution. As the effects of
these instructions on the nominal CPU state are eventually
reverted, we call them transient instructions. By influencing
which transient instructions are speculatively executed, we are
able to leak information from within the victim’s memory
address space.

**Attacks using Native Code**: As a proof-of-concept, we
create a simple victim program that contains secret data within
its memory address space. Next, we search the compiled
victim binary and the operating system’s shared libraries for
instruction sequences that can be used to leak information
from the victim’s address space. Finally, we write an attacker
program that exploits the CPU’s speculative execution feature
to execute the previously-found sequences as transient instructions. Using this technique, we are able to read memory from
the victim’s address space, including the secrets stored within
it.

**Attacks using JavaScript and eBPF**: In addition to violating
process isolation boundaries using native code, Spectre attacks
can also be used to violate sandboxing, e.g., by mounting
them via portable JavaScript code. Empirically demonstrating
this, we show a JavaScript program that successfully reads
data from the address space of the browser process running
it. In addition, we demonstrate attacks leveraging the eBPF
interpreter and JIT in Linux.


At a high level, Spectre attacks violate memory isolation boundaries by combining speculative execution with
data exfiltration via microarchitectural covert channels. More
specifically, to mount a Spectre attack, an attacker starts by
locating or introducing a sequence of instructions within the
process address space which, when executed, acts as a covert
channel transmitter that leaks the victim’s memory or register
contents. The attacker then tricks the CPU into speculatively
and erroneously executing this instruction sequence, thereby
leaking the victim’s information over the covert channel.
Finally, the attacker retrieves the victim’s information over
the covert channel. While the changes to the nominal CPU
state resulting from this erroneous speculative execution are
eventually reverted, previously leaked information or changes
to other microarchitectural states of the CPU, e.g., cache
contents, can survive nominal state reversion.

**Variant 1**: Exploiting Conditional Branches. In this variant
of Spectre attacks, the attacker mistrains the CPU’s branch
predictor into mispredicting the direction of a branch, causing
the CPU to temporarily violate program semantics by executing code that would not have been executed otherwise. As we
show, this incorrect speculative execution allows an attacker to
read secret information stored in the program’s address space.
Indeed, consider the following code example:
```bash 
if (x < array1_size)
y = array2[array1[x] * 4096];
```
In the example above, assume that the variable x contains
attacker-controlled data. To ensure the validity of the memory
access to array1, the above code contains an if statement
whose purpose is to verify that the value of x is within a
legal range. We show how an attacker can bypass this if
statement, thereby reading potentially secret data from the
process’s address space.

**Variant 2**: Exploiting Indirect Branches. Drawing from
return-oriented programming (ROP) [63], in this variant the
attacker chooses a gadget from the victim’s address space
and influences the victim to speculatively execute the gadget.
Unlike ROP, the attacker does not rely on a vulnerability in
the victim code. Instead, the attacker trains the Branch Target
Buffer (BTB) to mispredict a branch from an indirect branch
instruction to the address of the gadget, resulting in speculative
execution of the gadget. As before, while the effects of
incorrect speculative execution on the CPU’s nominal state are
eventually reverted, their effects on the cache are not, thereby
allowing the gadget to leak sensitive information via a cache
side channel. We empirically demonstrate this, and show how
careful gadget selection allows this method to read arbitrary
memory from the victim.
To mistrain the BTB, the attacker finds the virtual address
of the gadget in the victim’s address space, then performs
indirect branches to this address. This training is done from
the attacker’s address space. It does not matter what resides at
the gadget address in the attacker’s address space; all that is
required is that the attacker’s virtual addresses during training
match (or alias to) those of the victim. In fact, as long as the
attacker handles exceptions, the attack can work even if there
is no code mapped at the virtual address of the gadget in the
attacker’s address space.


