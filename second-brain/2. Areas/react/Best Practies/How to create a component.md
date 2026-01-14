component props
- hidden or display
- size
- flex or 
- align 
- justify
- padding
```jsx
<Button
  // 1. Reserved props
  ref={buttonRef}
  
  // 2. Styling / Standard attrs
  className="btn-primary"
  id="submit-main"
  
  // 3. Custom Data props
  label="Submit"
  variant="large"
  disabled={isLoading}
  
  // 4. Event Handlers
  onClick={handleSubmit}
  onMouseEnter={handleHover}
/>
```