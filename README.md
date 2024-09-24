# An unclickable button

What looks like a normal pushbutton - until the user tries to click on it.




![Sample Image - trick_button.gif](https://raw.githubusercontent.com/ChrisMaunder/trick_button/master/docs/assets/trick_button.gif)

## Introduction 

I wrote this many years ago for a friend. It is basically just a normal pushbutton, except that when the user tries to click on it, it moves out of the way making it practically un-clickable.  

I have no idea what anyone would ever want to use such a control in their app, apart from wanting to be extremely annoying to users (and hey - don't we all get like that on occasion?) :)

The button is just a normal button with `OnMouseMove` overridden: (`m_nJumpDistance` is the distance in pixels to jump once the mouse has moved over the control).

The `WM_SETFOCUS` message has also been handled to bounce the focus back to the window that previously had focus, and `PreSubclassWindow` has been overridden to allow the removal of the `WS_TABSTOP` window style bit. This ensures that the user can't tab to the control.

## The guts of it all

```cpp
void CTrickButton::OnMouseMove(UINT nFlags, CPoint point) 
{
    CWnd* pParent = GetParent();
    if (!pParent) pParent = GetDesktopWindow();

    CRect ParentRect;                                   // Parent client area (Parent coords)
    pParent->GetClientRect(ParentRect);

    ClientToScreen(&point);                             // Convert point to parent coords
    pParent->ScreenToClient(&point);

    CRect ButtonRect;                                   // Button Dimensions (Parent coords)
    GetWindowRect(ButtonRect);  
    pParent->ScreenToClient(ButtonRect);
    CPoint Center = ButtonRect.CenterPoint();           // Center of button (parent coords)

    CRect NewButtonRect = ButtonRect;                   // New position (parent coords)

    if (point.x > Center.x)                             // Mouse is right of center
    {
        if (ButtonRect.left > ParentRect.left + ButtonRect.Width() + m_nJumpDistance)
            NewButtonRect -= CSize(ButtonRect.right - point.x + m_nJumpDistance, 0);
        else
            NewButtonRect += CSize(point.x - ButtonRect.left + m_nJumpDistance, 0);
    }
    else if (point.x < Center.x)                       // Mouse is left of center
    {
        if (ButtonRect.right < ParentRect.right - ButtonRect.Width() - m_nJumpDistance)
            NewButtonRect += CSize(point.x - ButtonRect.left + m_nJumpDistance, 0);
        else
            NewButtonRect -= CSize(ButtonRect.right - point.x + m_nJumpDistance, 0);
    }

    if (point.y > Center.y)                           // Mouse is below center
    {
        if (ButtonRect.top > ParentRect.top + ButtonRect.Height() + m_nJumpDistance)
            NewButtonRect -= CSize(0, ButtonRect.bottom - point.y + m_nJumpDistance);
        else
            NewButtonRect += CSize(0, point.y - ButtonRect.top + m_nJumpDistance);
    }
    else if (point.y < Center.y)                      // Mouse is above center
    {
        if (ButtonRect.bottom < ParentRect.bottom - ButtonRect.Height() - m_nJumpDistance)
            NewButtonRect += CSize(0, point.y - ButtonRect.top + m_nJumpDistance);
        else
            NewButtonRect -= CSize(0, ButtonRect.bottom - point.y + m_nJumpDistance);
    }

    MoveWindow(NewButtonRect);
    RedrawWindow();
    
    CButton::OnMouseMove(nFlags, point);
}

void CTrickButton::OnSetFocus(CWnd* pOldWnd)
{
    CButton::OnSetFocus(pOldWnd);

    // Give the focus right back to the window that justn gave it to us
    if (pOldWnd!=NULL && ::IsWindow(pOldWnd->GetSafeHwnd()))
        pOldWnd->SetFocus();
}

void CTrickButton::PreSubclassWindow()
{
    // Ensure that the TabStop style is removed from the control
    ModifyStyle(WS_TABSTOP, 0);
    CButton::PreSubclassWindow();
}
```

## History

31 May 2002 - updated to include removal of `WM_TABSTOP`, plus minor cleanup.
