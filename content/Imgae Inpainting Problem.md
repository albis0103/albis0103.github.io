

**Define**
	:Reconstruct the missing or corrupt region from image
- Given image $I$ and binary mask $M$ (M=1:valid, M=0:missing)
	- Inpainting model Goal is learn Function $F$ let accord to $I_{masked}$ and $M$ estimate $\hat{I}$
	$I_{masked}=I \odot M$
	$\hat{I}=F(I_{masked}, M)=F(I \odot M, M)$ 



**Method**

- PDE-based
	:propagation Information from boundary inward, solve by PDE
		ex.Bertalmio Image Inpainting: Navier-Stokes likely function
- Patch-based
	:Assume Image has Self-similarity, find similar patch from image
		Criminisi Algo
			Priority Calculate: decide reconstruct order
				- $p$: 照度線(isophote)
				- $C(p)$:confidence, proportion of surround valid pixel
				- $D(p)$:data, represent structure idensity
					$D(p)=\nabla I_p^{\perp} \cdot \mathbf{n}_p$      
- Deep-Learn
	- Context - Encoder(Pathak): CNN(self-encoder), training by Reconstruct-loss and Adversarial-loss
	- PConv/ GConv
		[[CNN]]