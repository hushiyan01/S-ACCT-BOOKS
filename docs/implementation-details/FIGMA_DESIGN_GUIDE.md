# Figma Design Guide for S-ACCT-BOOKS

This document provides a structured guide for creating the Figma design files for the S-ACCT-BOOKS application.

## Figma File Structure

### Recommended Organization

```
📁 S-ACCT-BOOKS Design System
├── 📄 Cover Page (Overview, colors, links)
├── 📄 Design Tokens (Colors, Typography, Spacing, Shadows)
├── 📄 Components Library
├── 📄 Icons & Illustrations
└── 📄 Changelog

📁 S-ACCT-BOOKS Wireframes
├── 📄 User Flows
├── 📄 Web - Low Fidelity
├── 📄 Mobile - Low Fidelity
└── 📄 Annotations & Notes

📁 S-ACCT-BOOKS UI Designs
├── 📄 Web - Light Mode
├── 📄 Web - Dark Mode (optional)
├── 📄 Mobile - Light Mode
├── 📄 Mobile - Dark Mode (optional)
├── 📄 Tablet Layouts
└── 📄 States & Interactions

📁 S-ACCT-BOOKS Prototype
├── 📄 Web Prototype
└── 📄 Mobile Prototype
```

## Phase 1: Design System Setup

### 1. Color Styles

Create the following color variables in Figma:

**Primary Colors**
- `primary/main` - #1976D2
- `primary/light` - #42A5F5
- `primary/dark` - #1565C0
- `primary/contrast` - #FFFFFF

**Secondary Colors**
- `secondary/main` - #424242
- `secondary/light` - #616161
- `secondary/dark` - #212121
- `secondary/contrast` - #FFFFFF

**Semantic Colors**
- `success/main` - #4CAF50
- `success/light` - #81C784
- `success/dark` - #388E3C
- `error/main` - #F44336
- `error/light` - #E57373
- `error/dark` - #D32F2F
- `warning/main` - #FF9800
- `warning/light` - #FFB74D
- `warning/dark` - #F57C00
- `info/main` - #2196F3

**Neutral Colors**
- `background/default` - #FAFAFA
- `background/paper` - #FFFFFF
- `text/primary` - #212121
- `text/secondary` - #757575
- `text/disabled` - #BDBDBD
- `divider` - #E0E0E0
- `border` - #BDBDBD

**Financial Semantic Colors** (Custom)
- `financial/income` - #4CAF50 (use success/main)
- `financial/expense` - #F44336 (use error/main)
- `financial/neutral` - #616161

### 2. Typography Styles

Create text styles:

**Headings**
- `Heading/H1` - Inter/Roboto, Bold, 32px, line-height 40px
- `Heading/H2` - Inter/Roboto, Semi-Bold, 24px, line-height 32px
- `Heading/H3` - Inter/Roboto, Semi-Bold, 20px, line-height 28px
- `Heading/H4` - Inter/Roboto, Medium, 18px, line-height 24px

**Body**
- `Body/Large` - Inter/Roboto, Regular, 16px, line-height 24px
- `Body/Medium` - Inter/Roboto, Regular, 14px, line-height 20px
- `Body/Small` - Inter/Roboto, Regular, 12px, line-height 16px

**Special**
- `Caption` - Inter/Roboto, Regular, 12px, line-height 16px, color: text/secondary
- `Button` - Inter/Roboto, Medium, 14px, letter-spacing 0.5px, UPPERCASE
- `Overline` - Inter/Roboto, Regular, 10px, letter-spacing 1px, UPPERCASE

**Numbers/Amounts** (Important for financial app)
- `Amount/Large` - Inter/Roboto, Bold, 32px, line-height 40px, tabular-nums
- `Amount/Medium` - Inter/Roboto, Semi-Bold, 20px, line-height 28px, tabular-nums
- `Amount/Small` - Inter/Roboto, Medium, 14px, line-height 20px, tabular-nums

### 3. Spacing System

Create spacing variables:

- `spacing/4` - 4px
- `spacing/8` - 8px
- `spacing/12` - 12px
- `spacing/16` - 16px
- `spacing/24` - 24px
- `spacing/32` - 32px
- `spacing/48` - 48px
- `spacing/64` - 64px

### 4. Border Radius

- `radius/small` - 4px
- `radius/medium` - 8px
- `radius/large` - 16px
- `radius/pill` - 999px (for fully rounded)

### 5. Shadows (Elevation)

Based on Material Design elevation:

- `elevation/0` - None
- `elevation/1` - 0px 1px 3px rgba(0,0,0,0.12)
- `elevation/2` - 0px 2px 6px rgba(0,0,0,0.16)
- `elevation/3` - 0px 4px 12px rgba(0,0,0,0.16)
- `elevation/4` - 0px 8px 24px rgba(0,0,0,0.16)

## Phase 2: Component Library

### Core Components to Create

#### 1. Buttons

**Variants:**
- Type: Primary (filled), Secondary (outlined), Tertiary (text)
- Size: Small (32px), Medium (40px), Large (48px)
- State: Default, Hover, Active, Disabled
- Icon: None, Left, Right, Icon-only

**Properties:**
- Label (text)
- Full width (boolean)

#### 2. Input Fields

**Variants:**
- Type: Text, Number, Password, Email, Date
- Size: Small, Medium, Large
- State: Default, Focused, Error, Disabled, Filled

**Properties:**
- Label (text)
- Placeholder (text)
- Helper text (text)
- Error message (text)
- Required indicator (boolean)

#### 3. Dropdown/Select

**States:**
- Closed (default)
- Open (with menu)
- Selected
- Error
- Disabled

#### 4. Transaction Card

This is a custom component for the app.

**Variants:**
- Type: Expense, Income
- Size: Compact, Default, Detailed
- Interactive: View-only, Editable (with actions)

**Content:**
- Category icon (instance swap)
- Category name
- Transaction description
- Amount (with color coding)
- Date/time
- Shared badge (boolean)
- Action buttons (edit, delete)

#### 5. Balance Card

**Variants:**
- View: Personal, Group
- Period: Month, Year, All

**Content:**
- Balance amount (large)
- Trend indicator (up/down)
- Period selector
- Quick stats (income/expense)

#### 6. Category Icon Buttons

Create icon buttons for each category:
- Food & Dining (🍔)
- Transportation (🚗)
- Shopping (🛍️)
- Entertainment (🎬)
- Healthcare (⚕️)
- Education (📚)
- Bills & Utilities (💡)
- Salary (💼)
- Investment (📈)
- Other (•••)

**Variants:**
- State: Default, Selected, Disabled
- Size: Small, Medium, Large

#### 7. Date Picker

Material Design style date picker with:
- Month/Year selector
- Calendar grid
- Navigation arrows
- Today button
- Range selection (optional)

#### 8. Modal/Dialog

**Variants:**
- Size: Small (400px), Medium (600px), Large (800px), Full-screen (mobile)
- Actions: One button, Two buttons, Custom

**Structure:**
- Header (title + close button)
- Content area
- Footer (action buttons)

#### 9. Charts (Reference)

Create placeholder frames for charts:
- Pie chart mockup (category breakdown)
- Line chart mockup (trends)
- Bar chart mockup (comparisons)

Note: These will be implemented with actual chart libraries in code.

#### 10. Empty States

Create illustrations/icons for:
- No transactions yet
- No search results
- No group members
- Network error
- Loading state

## Phase 3: Screen Designs

### Design Each Screen in Multiple States

For each screen, create:
1. **Default state** - with realistic data
2. **Loading state** - skeleton screens or spinners
3. **Empty state** - no data yet
4. **Error state** - when something goes wrong
5. **Interaction states** - hover, active, etc.

### Responsive Design Approach

**Mobile-First Design**
1. Design mobile screens first (375px width - iPhone SE/standard)
2. Adapt to tablet (768px width - iPad)
3. Expand to desktop (1440px width - standard laptop)

**Key Breakpoint Frames:**
- Mobile: 375 x 812 (iPhone 13 Mini)
- Tablet: 768 x 1024 (iPad)
- Desktop: 1440 x 900 (MacBook)

## Phase 4: User Flow Diagrams

### Essential Flows to Visualize

Use FigJam or Figma's FigJam features to create:

#### 1. Authentication Flow
```
Landing Page
  ↓
Login ←→ Register
  ↓        ↓
Dashboard  Create/Join Group
             ↓
          Dashboard
```

#### 2. Add Transaction Flow
```
Dashboard
  ↓
Click "Add Expense" FAB
  ↓
Transaction Form
  ├─ Enter amount
  ├─ Select category
  ├─ Add description
  ├─ Choose date
  └─ Set visibility
  ↓
Save
  ↓
Success message → Return to Dashboard
```

#### 3. Group Management Flow
```
Settings
  ↓
Group Management
  ↓
Invite Member
  ├─ Email
  ├─ Invite Code
  └─ QR Code
  ↓
Send Invitation
```

#### 4. Report Generation Flow
```
Reports Tab
  ↓
Select Filters
  ├─ Date range
  ├─ Personal/Group
  └─ Categories
  ↓
Generate Report
  ↓
View Charts
  ↓
Export (optional)
```

## Phase 5: Prototyping

### Interactive Prototype Requirements

#### Web Prototype
- Start: Login screen
- Key interactions:
  - Login → Dashboard
  - Add transaction (full flow)
  - View transaction details
  - Apply filters
  - Navigate between tabs
  - View reports

#### Mobile Prototype
- Start: Splash screen → Login
- Key interactions:
  - Login → Dashboard
  - FAB → Add transaction
  - Swipe transaction card actions
  - Bottom navigation
  - Pull to refresh
  - View reports

### Micro-interactions to Show

1. **Button states:** Ripple effect on click
2. **Input focus:** Underline animation
3. **Modal:** Fade in/scale animation
4. **Transaction card:** Swipe to reveal actions (mobile)
5. **Success feedback:** Checkmark animation
6. **Loading:** Skeleton screens or shimmer effect
7. **Page transitions:** Slide or fade

## Phase 6: Design QA Checklist

Before handoff to developers:

### Consistency Check
- [ ] All screens use design system colors
- [ ] Typography is consistent across screens
- [ ] Spacing follows 8px grid system
- [ ] Components are properly documented
- [ ] All variants are created

### Completeness Check
- [ ] All screens in product spec are designed
- [ ] All user flows are complete
- [ ] All states are designed (default, loading, empty, error)
- [ ] Responsive designs for mobile, tablet, desktop
- [ ] Icons and illustrations are included

### Accessibility Check
- [ ] Color contrast meets WCAG 2.1 AA standards (use plugin)
- [ ] Touch targets are minimum 44x44px
- [ ] Text is readable at all sizes
- [ ] Focus states are visible
- [ ] Form fields have clear labels

### Developer Handoff
- [ ] Design tokens exported as JSON
- [ ] All assets exported at @1x, @2x, @3x (mobile)
- [ ] SVG icons exported
- [ ] Annotations for complex interactions
- [ ] Component properties documented
- [ ] Prototype links shared

## Recommended Figma Plugins

### Design
- **Material Design Icons** - Icon library
- **Iconify** - Massive icon collection
- **Unsplash** - Stock photos for realistic mockups
- **Content Reel** - Generate realistic content

### Prototyping
- **Stark** - Accessibility checker (contrast, focus order)
- **Figmotion** - Advanced animations
- **Autoflow** - Generate flow diagrams

### Handoff
- **Design Tokens** - Export design tokens as JSON
- **Figma to Code** - Generate code snippets
- **Measure** - Show spacing and dimensions
- **Redlines** - Add measurement annotations

## Design System Export Format

Export design tokens as JSON for developers:

```json
{
  "colors": {
    "primary": {
      "main": "#1976D2",
      "light": "#42A5F5",
      "dark": "#1565C0"
    },
    "semantic": {
      "income": "#4CAF50",
      "expense": "#F44336"
    }
  },
  "typography": {
    "fontFamily": "Inter",
    "h1": {
      "fontSize": "32px",
      "fontWeight": "700",
      "lineHeight": "40px"
    }
  },
  "spacing": {
    "xs": "8px",
    "sm": "12px",
    "md": "16px",
    "lg": "24px"
  }
}
```

## Timeline Estimate

- **Week 1:** Design system setup + component library
- **Week 2:** Wireframes for all core screens
- **Week 3:** High-fidelity designs (mobile)
- **Week 4:** High-fidelity designs (web) + responsive
- **Week 5:** Prototype + interactions
- **Week 6:** Review, revisions, handoff

## References & Inspiration

### Apps with Similar UX
- **Splitwise** - Shared expense tracking
- **Money Lover** - Personal finance
- **Wallet by BudgetBakers** - Multi-currency
- **Spendee** - Category visualization
- **Goodbudget** - Envelope budgeting

### Design Systems to Reference
- **Material Design 3** - https://m3.material.io
- **Ant Design** - https://ant.design
- **Carbon Design System** - https://carbondesignsystem.com

### Figma Community Resources
- Search "personal finance" or "expense tracker" for templates
- Material Design 3 UI Kit
- Financial app icon sets

## Questions for Designer

Before starting, please clarify:

1. **Brand Identity:** Do you have specific brand colors/logo, or should we use the proposed palette?
2. **Illustrations:** Should we use illustrations for empty states, or keep it minimal?
3. **Dark Mode:** Priority for MVP or Phase 2?
4. **Animation Complexity:** Simple transitions or elaborate micro-interactions?
5. **Chart Style:** Colorful and vibrant or professional and minimal?
6. **Target Audience Preference:** Younger (modern, playful) or older (traditional, clear)?

---

**Next Step:** Use this guide to create the Figma design files. Start with the design system, then wireframes, then high-fidelity designs.
