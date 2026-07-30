
### LSTM:RNN + 3 gate( for maintenance Cell state)

create the passageway , let controlled the gateway to retain wondering the $h_t$ to output(not encounter gradient descent)
![[Pasted image 20260718143340.png]]

**Forget gate 
	$f_t$ : decides what information need to throw, written into $C_t$**, used $x_t$ and $h_{t-1}$ to learned $W_f$
		$f_t=\sigma(W_f \cdot [h_{t-1}, x_t]+b_f)$ 
		- '1' : keep this
		- '0' : get rid this
**Input gate
	$i_t$  decides what information going to store in cell state**
		$i_t=\sigma(W_i \cdot [h_{t-1},x_t]+b_i)$
	**$\tilde{C_t}$ : Vanilla RNN likely, process new information and hidden state in time $t$ 
		$\tilde{C}=tanh(W_C\cdot [h_{t-1}, x_t]+b_c)$ 
Cell state update(Long term memory): used Forget gate $f_t$and Input gate $i_t$ to control
	$C_t$ :forget we decide forget($f_tC_{t-1}$), update new Candidate(New) values($i_t\tilde{C_t}$)
	$C_t=f_t \cdot C_{t-1}+i_t \cdot \tilde{C_t}$ 
	![[Pasted image 20260718143406.png]]


**Output gate**
**: decides that new information output, readout of $C_t$** 
	$o_t=\sigma(W_o  \cdot [h_{t-1},x_t]+b_i)$
	**Hidden state (Short term memory)**
	$h_t$ : used current output $o_t$ to decide how to used memory(cell state $tanh(C_t)$)
	$h_t=o_t \cdot tanh(C_t)$ 


### Variant of LSTM

- LSTM with peephole Connection
	: allow each gate look cell state
	resolve when $C_{t-1}=5, o_{t-1}=0$ ,cause $h_{t-1}=0$ , can not separate is $C_{t-1}=0$ or $o_{t-1}=0$
	- $f_t=\sigma(W_f \cdot [C_{t-1}, h_{t-1}, x_t]+b_f)$
	- $i_t=\sigma(W_i \cdot [C_{t-1}, h_{t-1}, x_t]+b_i)$
	- $o_t=\sigma(W_o \cdot [C_{t-1}, h_{t-1}, x_t]+b_o)$ 
	[[ConvLSTM(Convolution LSTM)]]
- LSTM with Coupled Forget/Input Gates
	: let $f_t$ decide what information need to forget and store in cell state , instead of separately gate $(f_t, i_t) \rightarrow (f_t, 1-f_t)$ 
		$C_t = f_t \cdot C_{t-1} + (1-f_t) \cdot \tilde{C_t}$ 