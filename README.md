# ED_FullSpace
## Requirements
Fortran(2003) + lapack
## How to use it?
Follow the steps in /Install.
## Introduction
One should first have a knowledge of what is [ED_Subspace](https://github.com/HengyueLi/ED_Subspace) object. This module is an extension of that.  </br>&nbsp;
ED_FullSpace is a Quantum Cluster Solver for fermion spin 1/2 system. It considers a Grand Canonical Ensemble(GCE) system. You can use this code to study a quantum cluster model(Hubbard model for instance) within several tens of lines of code. All kinds of interactions can be considered. The symmetry ( [H,N] = 0 , [H,Sz]=0) of the system can be considered to increase the speed and save the memory. Compare with [LanczosGCE](https://github.com/HengyueLi/LanczosGCE/blob/master/README.md), this module can do the same calculation at finite temperature.
## example
Here is a complete example (with only (around) 20 statements in the main program) to solve a two-site Hubbard system. The ground state energy and degeneracy are shown. The system contains a hopping term and onsite Coulomb energy.

    program main
      use ED_FullSpace
      implicit none

       TYPE(ED_GCE)::ED
       type(table)     :: Ta
       type(Ham)       :: H

        integer::hpara(8)
        complex*16::v
        character*16::Intp


        ! Initialization of table   (see https://github.com/HengyueLi/Fermion_Table )
        ! In this example, the real symmetry of the system is 2. So one can use symmetry = 0/1/2
        ! symmetry = 0 / 1 / 2 dose not change the final result and all the calculation.
        ! But one should remenber that the higher value it is, the faster the calculation will be.(And
        ! also the smaller memorry it will use. How different it is? -> Very different! )
        call ta%Initialization( ns = 2 ,  symmetry = 2 ) ! check = 0 , 1 , 2




        ! Define the Hamiltonian of the system (see https://github.com/HengyueLi/FermionHamiltonian )
        call H%Initialization( ns = 2 )
        ! start append H terms  (needed)
        call h%StartAppendingInteraction()



        !! set the hopping term
        Intp     = "Hopping"
        hpara(1) = 0
        hpara(2) = 1
        v        = (1.0_8,0._8)
        call h%AppendingInteraction(InterType=Intp,InterPara=hpara,InterV=v)


        !! Set onsite Coulomb energy
        Intp     = "GlobalOnSiteU"
        v        = (4.0_8,0._8)
        call h%AppendingInteraction(InterType=Intp,InterPara=hpara,InterV=v)


        !Finished setting Hamiltonian
        call h%EndAppendingInteraction()


        ! Initialize the ED solver
        ! IsReal_ is optional
        ! Notice that here we have introduced the temperature T. Because in this module,
        ! some thermal quantities (partition function for instance) can be calculated where the
        ! temperature is needed ( the factor EXP(-\beta * E) ).
        Call ED%Initialization(T=0.1_8,Ta=Ta,CH=H)
        Call ED%Initialization(T=0.1_8,Ta=Ta,CH=H,IsReal = .True. )



        ! diagonalization
        Call ED%diagonalization()

        !-----If one do not care about thermal quantities, the calculation is done.
        ! If we need further to calculate that, we need to call the function below to get ready
        Call ED%Normolize_Energy_And_GetZ()

        ! If Normolize_Energy_And_GetZ() is called, all the eigenvalue would be shifted by
        ! ground state energy. E = E - Eg.

        ! show Eg
        write(*,*)"The ground state energy is:",ed%get_Eg()

        ! partition funciton
        write(*,*)"The partition funciton is:",ed%GetRz()

        ! The two-sites system can be solved analytically. One can check that himself.

        ! More useful functions can be found in source code directly.

    end
