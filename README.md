## Flowter
A lightweight, Swifty and customizable UIViewController flow cordinator

### Install
#### Carthage
Just add the entry to your Cartfile:
```
github "zazcarmobile/Flowter"
```


#### CocoaPods
To be implented...

### Basic usage
Create the flow and set its UIViewControllers and a Containter.
The Flowter can be created anywhere, regarding that you have a reference to who will present your flow, just use it on the startFlow closure:
```swift
        Flowter(with: UINavigationController())
            .addStep { $0.make(with: StepViewController(withLabel: "1st Step"))}
            .addStep { $0.make(with: StepViewController(withLabel: "2nd Step"))}
            .addStep { $0.make(with: StepViewController(withLabel: "3rd Step"))}
            .addEndFlowStep { (container) in
                container.dismiss(animated: true)
            }
            .startFlow { (container) in
                myNavigationController.present(container, animated: true)
        }
```

Your have to make your controllers conforming to the protocol Flowtable.
This only specify that your controllers have an var named flow of the type FlowStepInfo?
```swift
    var flow: FlowStepInfo?
```

You call this on the controller when it is ready to proceed with the flow:
```swift
    private func nextStep() {
        flow?.next()
    }
```

### Dependency injection
You can fully customize the factory clousure of each of your step UIViewController subclass, at this moment you can feed it with it needings.
```swift
        let newUser = User()
        
        Flowter(with: UINavigationController())
            .addStep { $0.make(with: FirstStepViewController(withUser: newUser))}
            .addStep { $0.make(with: {
                let viewModel = SecondStepViewModel(with: newUser)
                return SecondStepViewController(with: viewModel)
             )}
            .addStep(with: { (stepFactory) -> FlowStep<ThirdStepViewController, UINavigationController> in
                let step = stepFactory.make(with: ThirdStepViewController())
                step.setPresentAction({ (thirdStepVC, container) in
                    thirdStepVC.setUser(newUser)
                    thirdStepVC.setCustomParameter(foo: false)
                    container.pushViewController(thirdStepVC, animated: false)
                })
                return step
            })
            .addEndFlowStep { (container) in
                container.dismiss(animated: true)
            }
            .startFlow { [weak self] (container) in
                self?.present(container, animated: true)
        }
```
You can do every thing you could do inside your viewControllers, in an easy to visualize and flexible sintax.

### Custom presentation and dismiss code
#### Presentation:
You have control of the presentation code of your controllers too! 
```swift
            .addStep(with: { (stepFactory) -> FlowStep<StepViewController, UINavigationController> in
                let step = stepFactory.make(with: StepViewController(withLabel: "Flow Start"))
                step.setPresentAction({ (welcomeVC, container) in
                    welcomeVC.setAsWelcomeStep()
                    container.pushViewController(welcomeVC, animated: false) //I don't known why I would do this...
                })
                return step
            })
```

#### Dissmiss:
The dismiss is also avaliable when you view controllers dont rely on the automatic UINavigationController back button.

You have to call the FlowStepInfo method back to navigate back, your step dismissAction closure will be called and you will be responsable for the dismiss of the viewController
```swift
            .addStep(with: { (stepFactory) -> FlowStep<StepViewController, UINavigationController> in
                let step = stepFactory.make(with: StepViewController(withLabel: "Flow Start"))
                step.setPresentAction({ (welcomeVC, container) in
                    container.pushViewController(welcomeVC, animated: false) //I don't known why I would do this...
                })
                
                step.setDismissAction({ (welcomeVC, container) in
                    container.dismiss(animated: false, completion: {
                      //some completion code
                    })
                })

                return step
            })
```

On your Flowtable conforming UIViewController subclass:
```swift
    private func backStep() {
        flow?.back()
    }
    
    private func nextStep() {
        flow?.next()
    }
```