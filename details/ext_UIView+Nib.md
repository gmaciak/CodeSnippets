# Ext: UIView+Nib.swift
- **shortcut**: `ext_UIView+Nib`
- **language**: Swift
- **platform**: iphoneos

## Summary
Support for loading and instantiating views from Nib (Xib)

## Code:
```swift
//
//  UIView+Nib.swift
//
//
//  Created by Grzegorz Maciak on 21.06.2018.
//  Copyright (c) 2018 Grzegorz Maciak.
//  This code is distributed under the terms and conditions of the MIT license.
//  See: https://gist.github.com/kodelit/04b5ef889e23f63f3462c8595d0164d9#file-license-md

import Foundation

// MARK: - View instance from Nib

public protocol ViewInstanceFromNib:AnyObject {
    static var nib:UINib { get }
    static func instantiate(nib:UINib?) -> Self?
}

public extension ViewInstanceFromNib where Self:UIView
{
    public static var nib:UINib {
        return UINib(nibName: NSStringFromClass(self).components(separatedBy: ".").last!, bundle: Bundle(for: self))
    }
    
    public static func instantiate(nib:UINib? = nil) -> Self? {
        if let view = (nib ?? self.nib).instantiate(withOwner: nil, options: nil).first! as? Self {
            return view
        }
        return nil
    }
}

public extension ViewInstanceFromNib where Self:UITableViewCell
{
    static var defaultReuseId:String {
        return NSStringFromClass(self).components(separatedBy: ".").last!
    }
}

public extension ViewInstanceFromNib where Self:UITableViewHeaderFooterView
{
    static var defaultReuseId:String {
        return NSStringFromClass(self).components(separatedBy: ".").last!
    }
}

extension UIView: ViewInstanceFromNib {}

// MARK: - View loading from Nib (for IB)

public protocol NIBRepresentable:AnyObject {
    var contentView: UIView? { get set }
    var customNibName:String? { get }
    
    init(frame: CGRect)
    init?(coder aDecoder: NSCoder)
    func commonInitFromNib() // might but in most cases default implementation should not be overriden
}

public extension NIBRepresentable where Self:UIView {
    public var customNibName:String? { return nil }
    
    public func commonInitFromNib() {
        if contentView == nil {
            if let nibName = self.customNibName {
                let nib = UINib(nibName: nibName, bundle: Bundle(for: type(of: self)))
                nib.instantiate(withOwner: self, options: nil)
            }else{
                Self.nib.instantiate(withOwner: self, options: nil)
            }
            
            if let view = contentView {
                view.frame = self.bounds
                view.translatesAutoresizingMaskIntoConstraints = false
                self.addSubview(view)
                
                NSLayoutConstraint.activate(NSLayoutConstraint.constraints(withVisualFormat: "H:|[view]|", options: [], metrics: nil, views: ["view": view]))
                NSLayoutConstraint.activate(NSLayoutConstraint.constraints(withVisualFormat: "V:|[view]|", options: [], metrics: nil, views: ["view": view]))
            }
        }
    }
}


// MARK: - Example

//@IBDesignable
//class CustomView: UIView, NIBRepresentable {
//    @IBOutlet public var contentView: UIView?
//
//    @IBInspectable public var nibName:String? = nil
//    open var customNibName:String? { return nibName }
//
//    public required override init(frame: CGRect) {
//        super.init(frame: frame)
//        self.commonInitFromNib()
//    }
//
//    public required init?(coder aDecoder: NSCoder) {
//        super.init(coder: aDecoder)
//        self.commonInitFromNib()
//    }
//
//    override var intrinsicContentSize: CGSize {
//        return CGSize(width: UIViewNoIntrinsicMetric, height: UIViewNoIntrinsicMetric)
//    }
//}
```