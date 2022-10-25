# What is SnapKit:

**SnapKit** is a DSL wrapper over Auto Layout that allows you to lay out views with code, without xibs and storyboards.
**More details [here](https://github.com/SnapKit/SnapKit) and [here](http://snapkit.io/docs/).**

**Pros of this approach:**
- Fast and clean writing UIView
- No conflicts in nib files when merging branches in git
- Ability to conduct a code-review (and not look at an auto-generated xml file)
- Rejection of magic numbers constraints in favor of grid
- Easier to reuse items
- Everything in one place: no need to constantly switch between storyboard and code for element configuration

# The main thing from the example (see below):

- Protocol **[Grid](Sources/Grid.swift)** with **private extension**, where all base padding values ​​are located, as well as custom padding if needed (ex: space22)
- Protocol **[Appearance](Sources/Appearance.swift)** with **private extension**, where there are various magic numbers and custom fonts/colors that don't need to be added globally. In general, everything related to **UI** specific view
- Structure **Constants** with **private extension** where numeric and text constants are located
- UI elements are placed visually in a separate block on top of the class
- UI elements are initialized and configured nicely with [**Then**](https://github.com/devxoul/Then), preferably via **lazy computed property**
- There are two **separate** functions **addSubviews()** and **makeConstraints()**
- In the **addSubviews()** function, child views are added in [**hierarchical form**](Sources/UIView%2BAdd.swift)

# An example of a regular UIView:

```swift

// MARK: - Grid

private extension Grid {
    /// heightRow
    var space60: CGFloat { 60 }
    /// Top padding emailTextField
    var space22: CGFloat { 22 }
    /// loginButton height
    var space55: CGFloat { 55 }
}

// MARK: -Appearance

private extension Appearance {
    var animationDuration: Double { 0.1 }
    varparallaxValue: CGFloat { 10 }
    var alphaContainerView: CGFloat { 0.5 }
    var borderColor: UIColor { .customColor }
    var customFont: UIFont { .customFont }
    var buttonCornerRadius: CGFloat { 10 }
}

// MARK: - Constants

private extension Constants {
    static let emailPlaceholder = "Enter an email"
    static let textCharactersLimit = 140
}

/// Login screen using email and password
final class LoginView: UIView {
       
    // MARK: - Private Properties
    
    private lazy var emailTextField = UITextField().then {
        $0.textContentType = .emailAddress
        $0.keyboardType = .emailAddress
        $0.placeholder = Constants.emailPlaceholder
    }
    
    private lazy var passwordTextField = UITextField().then {
        $0.keyboardType = .password
        $0.borderColor = appearance.borderColor
    }
    
    private lazy var loginButton = UIButton().then {
        $0.backgroundColor = .scDeepBlush
        $0.cornerRadius = appearance.buttonCornerRadius
    }
    
    // MARK: - UIView
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        commonInit()
    }
    
    // MARK: - Private Methods

    private func commonInit() {
        setupStyle()
        addSubviews()
        makeConstraints()
    }
    
    private func setupStyle() {
        backgroundColor = .white
    }
    
    private func addSubviews() {
        add {
            contentView.add {
                emailTextField
                passwordTextField
                loginButton
            }
        }
    }
    
    private func makeConstraints() {
        contentView.snp.makeConstraints { make in
           make.edges.equalToSuperview().inset(grid.space16)
        }
        
        emailTextField.snp.makeConstraints { make in
           make.top.equalToSuperview().inset(grid.space22)
           make.leading.trailing.equalToSuperview().inset(grid.space16)
        }
        
        passwordTextField.snp.makeConstraints { make in
            make.top.equalTo(emailTextField.snp.bottom)
            make.leading.trailing.equalToSuperview().inset(grid.space16)
        }
        
        loginButton.snp.makeConstraints { make in
            make.bottom.leading.trailing.equalToSuperview().inset(grid.space8)
            make.height.equalTo(grid.space55)
        }
    }
    
}

```

# Tips and rules for layout using SnapKit:

**one. Write leading and trailing instead of left and right**

**2. Describe the constraints in priority order: vertical (top to bottom), horizontal (left to right), size**

```
top, bottom, centerY, leading, trailing, centerX, width, height, size
```

**3. Combine all similar indents together**

✅ Right
```swift
make.top.leading.trailing.equalToSuperView().inset(grid.space16)

```
❌ Wrong
```swift
make.top.equalToSuperView().inset(grid.space16)
make.leading.equalToSuperView().inset(grid.space16)
make.trailing.equalToSuperView().inset(grid.space16)
```

**four. In any unclear situation, use inset **

✅ Exception: If it is an indent between two elements: leading to trailing. bottom to top

**5. We use the shortest notation: edges**

✅ Right
```swift
make.edges.equalToSuperView()
```

❌ Wrong
```swift
make.top.bottom.leading.trailing.equalToSuperView()
```

**5. We use the shortest notation: center**

✅ Right
```swift
make.center.equalToSuperView()
```

❌ Wrong
```swift
make.centerX.centerY.equalToSuperView()
```

**6. We use the shortest notation: size**

✅ Right
```swift
make.size.equalTo(grid.size50)
```

❌ Wrong
```swift
make.width.height.equalTo(grid.size50)
```

**7. Parent views don't need to know anything about child view constraints:**

✅ Right
```swift
rootView.addSubview(childView)
...
childView.snp.makeConstraints { make in
    make.centerX.equalToSuperview()
}

```

❌ Wrong
```swift
rootView.addSubview(childView)
...
rootView.snp.makeConstraints { make in
    make.centerX.equalTo(childView)
}
```
