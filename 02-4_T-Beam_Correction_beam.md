## Table of content

[General](#General)

[Download File](#Download%20File)

[What does the Software do?](#What%20does%20the%20Software%20do?)

[Materials](#Materials)

[Cross sections](#Cross%20sections)

[System](#System)

[Actions / Loads](#Actions%20/%20Loads)

[Linear calculation](#Linear%20calculation)

[Results and Discussion](#Results%20and%20discussion)

[Special cases](#Special%20cases)

[Pros & Cons](#Pros%20&%20Cons)

## General

Used finite elements:
- Beam Element with T-Beam cross section
- Quad Elements
- Correction beam - dummy beam made by the software as a beam element on the position of the slab with negative weight and negative E-Modulus

Only beam elements visualized:
![[T-Beam_Approach-3_Visual_Beam.png]]

Only quad / plate elements visualized:
![[T-Beam_Approach-3_Visual_slab.png]]

## Download File

The Sofistik Teddy File can be downloaded here:
[T-Beam Example Correction Beam](https://github.com/AIztok/Modelling-Analysis_Structural_Concrete/blob/main/SOFiSTiK_Files/02_T-beam/T-beam_Example-Correction_Beam.dat)

## What does the Software do?

The software performs in the background the following operations:

- no direct siffness or mass adaptaion of the beam or quads
- in the global stiffness matrix of the system a fictional "correction beam" with negative stiffness and mass is added
- calculation of the internal forces (beam, correction beam and quads), addition of the beam and correction beam results

Graphic representation of the procedure (©SOFiSTiK):

![[T-Beam_Approach-4_Correction-beam_01.png]]

![[T-Beam_Approach-4_Correction-beam_02.png]]
 
## Materials

Teddy Input for defining the materials:

```
+prog aqua
head 'Materials'
$----------------------------------------------------------------------------------
!*! Specification of the standard
NORM OEN en199X-200X unit 0 !
$----------------------------------------------------------------------------------
!*! Scope of output
echo full extr
$----------------------------------------------------------------------------------
!*! Concrete
CONC NO 1  TYPE c 30N           titl 'Conc. C30/37'

$----------------------------------------------------------------------------------
!*! Reinforcement
STEE NO 101 TYPE b 550b titl 'Reinf. B550B'
          
end 
```
## Cross sections

Teddy Input for defining the cross section of the beam elements:

```
+prog aqua
head 'Cross sections'
$----------------------------------------------------------------------------------
!*! Units for input and output (i = input; o = output)
page unii 0 unio 0

$----------------------------------------------------------------------------------
!*! Control
ctrl rest 2 ! Control to ensure the specifications of the previous AQUA are not overwritten

$----------------------------------------------------------------------------------
!*! T-beam - with polygon points
SECT NO  1  MNO  1 MRF 101 titl 'T-beam Poly'

let#h  0.80[m] ! Variable for cross section height
let#b  0.30[m] ! Variable for cross section width
let#ho 0.20[m] ! Variable for top flange thickness
let#bo 2.50[m] ! Variable for top flange width

!*! Stress points - where the stresses will be calculated
$ Top edge
SPT OKL  Y -#bo/2 Z  0 MNO 1
SPT OKM  Y  0.00  Z  0 MNO 1
SPT OKR  Y +#bo/2 Z  0 MNO 1
$ Bottom edge
SPT UKL  Y -#b/2  Z #H MNO 1
SPT UK   Y  0.00  Z #H MNO 1
SPT UKR  Y +#b/2  Z #H MNO 1
$ Polygon points
POLY TYPE O   MNO  1
    VERT   1  Y    0     Z   0       REFP OKL
    VERT   2  Y    0     Z   0       REFP OKM
    VERT   3  Y    0     Z   0       REFP OKR
    VERT   4  Y    0     Z   #ho     REFP OKR
    VERT   5  Y  #b/2    Z   #ho     REFP OKM
    VERT  11  Y    0     Z   0       REFP UKR
    VERT  12  Y    0     Z   0       REFP UK
    VERT  13  Y    0     Z   0       REFP UKL
    VERT  15  Y -#b/2    Z   #ho     REFP OKM
    VERT  14  Y    0     Z   #ho     REFP OKL

! Shear cuts
CUT 'Csc' ZB 'S' ! shear cut through the centre of gravity
CUT 'Web' ZB #ho+0.01 ! shear cut below the slab
CUT 'Fla' YB #b/2+0.01 ZB +0.5 REFA 2 YE #b/2+0.01 ZE -0.05 REFE 2
         
end 
```

## System

Input of the static system for the automatic meshing using the `SOFiMSHC` Module.

In the Figure below the Structural Node and Structural Line Numbers are shown:

![[T-Beam_Approach-3_Visual_SPT&SLN.png]]

SOFiMSHC Code Block:

```
+prog sofimshc
head 'Model'
$----------------------------------------------------------------------------------
!*! System and mesh parameters
ctrl mesh 2 $ 1 - bar model, 2 - surface model, 3 - solid model
ctrl hmin 0.25 $[m] Size of finite elements

syst 3D gdir posz gdiv 10000
$ gdir: global coordinate system. posz = Z-direction downward
$ gdiv: group divisor = how many finite elements in a group (with 100, the elements of group 1 can be: 101, 102 up to 199)

$----------------------------------------------------------------------------------
!*! Structural points
spt no  x       y       z               fix
    1   0       0       0               -
    11  0       2.5     0               -
    2   5       0       0               -
    12  5       2.5     0               -
    21  0       1.25    0               pp
    22  5       1.25    0               py
    31  0       1.25    0.1/2+0.2/2     -
    32  5       1.25    0.1/2+0.2/2     -
$ Larger span length
$spt no  x       y       z               fix
$    1   0       0       0               -
$    11  0       2.5     0               -
$    2   15       0       0               -
$    12  15       2.5     0               -
$    21  0       1.25    0               pp
$    22  15       1.25    0               py
$    31  0       1.25    0.1/2+0.2/2     -
$    32  15       1.25    0.1/2+0.2/2     -

$----------------------------------------------------------------------------------
!*! Structural lines
sln 1 1  11 fix pz
sln 2 11 12
sln 3 12 2  fix pz
sln 4 2  1
$ Beam
sln 20 21 22 sno 1 grp 1

$----------------------------------------------------------------------------------
!*! Structural areas
$ Slab
sar 1 grp 2 mno 1 mrf 101 t 0.20[m] qref belo mctl regm drx 1 0 0
    sarb out nl 1,2,3,4
    
       
end   
```


## Actions / Loads

As no combinations acc. Standards are created, only the loads with no assignment to actions are defined:

```
+prog sofiload
head 'Lasten'

$ Deadload
lc 1 facd 1.0 titl 'EGW'
$ Additional permanent load
lc 2 titl '5kN/m2'
    quad grp 2 type pg p 5

end    
```


Only the T-Beam has a weight:

![[T-Beam_Approach-3_DL.png]]

## Linear calculation

Firstly the T-Beam Approach 4 is activated, all further `ASE` Moduls are calculated with the TBEX activated:

```
+prog ase
head 'T-Beam activated'
$----------------------------------------------------------------------------------
!*! Scope of output
echo full extr
page lano 1 $ Output in 0 = German / 1 = English

$----------------------------------------------------------------------------------
!*! T-Beam correction beam activated
TBEX NOG AUTO

end   
```

Linear calculation is performed as following, here both load cases are calculated separately and once a load combination is created within ASE Module:

```
+prog ase
head 'Calculation'
$----------------------------------------------------------------------------------
!*! Scope of output
echo full extr
page lano 1 $ Output in 0 = German / 1 = English

$----------------------------------------------------------------------------------
!*! Analysis parameters
syst prob line $ linear analysis

$----------------------------------------------------------------------------------
!*! Element groups
grp - $ activate all element groups

$----------------------------------------------------------------------------------
!*! Load cases
lc all $ calculate all load cases defined in SOFiLOAD

$ Example direct load combination in ASE
lc 201 facd 1.35 $ Self-weight cannot be copied; it must be defined explicitly in ASE if used with other copied loadcases (lcc)
lcc 2 fact 1.35 $ Load case 2 is copied and factored
     
end  
```

## Results and Discussion

The results can be shown using `Graphic`, below the code block is given to generate the shown plots directly from `TEDDY`.

Internal forces of the T-beam:
- Bending moment M<sub>y</sub>
- Shear forces V<sub>z</sub>
- Normal force N (should be zero)

```
+PROG WING
HEAD 'Results beam'

SIZE TYPE URS SC 0 MARG NO SPLI '3*1'

VIEW TYPE DIRE X 0 Y -1 Z 0 AXIS POSZ ROTA 0

DSGN TYPE LINE DTYP DEFA
BEAM TYPE DSGN ROPT P STAT ALL STAL ALL OFFZ MIDD

LC   NO 1 ; BEAM TYPE MY UNIT DEFA SCHH 0.5 STYP BEAM FILL NO REPR DLIN
LC   NO 1 ; BEAM TYPE VZ UNIT DEFA SCHH 0.5 STYP BEAM FILL NO REPR DLIN
LC   NO 1 ; BEAM TYPE N UNIT DEFA SCHH 0.5 STYP BEAM FILL NO REPR DLIN

END 
```

![[T-Beam_Approach-4_internal-f_beam.png]]

As we see, the results in the beam don't corespond good with the other approaches, the bending moment is larger (60,7 kNm > 53 kNm) and there is a compression force in the beam.
The reason is, that if the normal forces and bending moment of the slab is checked, the shear lag effect is large, e.g. the normal force is not constant over the width but there are large deviation. Wheres in a correction beam the shear lag can't be considered (constant axial stiffness over the width), therefore the internal forces in the correction beam are larges as those in the slab.

![[T-Beam_Approach-4_Internal-f_slab.png]]

The reason is the span to width ratio of the model (5 m / 2,5 m = 2 / 1). If this ratio is increase to 15 m / 2,5 m = 6 /1 (see in the chapter [[#System]]) to uncomment the 15 m SPTs. Then this effects gets much smaller and the results are very near to other approaches.

## Special cases 

/

## Pros & Cons

The pros of this approach are:
- simple modelling as the software covers the adaptation of mass / stiffness in the background
- design of slab possible in the same model

The cons of this approach are:
- If shear lag effect large, the results might deviate from the correct results.


