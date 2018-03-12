  375  git diff HEAD^^^..HEAD --stat
    376  git diff HEAD^^^..HEAD -p
      377  git diff HEAD^^^..HEAD > patch.diff
        378  vim patch.diff 
          379  git branch 
            380  ls
              381  pwd
                382  git status
                  383  git add arch/x86/kernel/cpu/common.c arch/x86/kernel/cpu/transmeta.c arch/x86/kvm/vmx.c arch/x86/lguest/boot.c
                    384  git commit -m "working"
                      385  mv patch.diff ../
                        386  git log
                          387  git checkout 5f80d0e7906f7dfc4a9dd946b486435c00383d99 -b patching
                            388  git log
                              389  git status
                                390  git apply ../patch.diff
                                  391  git status
                                    392  history
