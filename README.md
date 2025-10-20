# Expenseura - Smart Expense Management System

A full-stack expense management application built with React and Supabase, featuring role-based authentication, multi-level approvals, OCR receipt extraction, and multi-currency support.

## Features

### Core Functionality
- **Role-Based Access Control**: Admin, Manager, and Employee roles with specific permissions
- **Multi-Level Approvals**: Configurable approval workflows with sequential or parallel processing
- **OCR Receipt Extraction**: Automatic data extraction from receipt images using Tesseract.js
- **Multi-Currency Support**: Automatic currency conversion to company base currency
- **Receipt Storage**: Secure file storage with Supabase Storage

### User Roles

#### Admin
- Create and manage company users
- Configure approval rules (percentage, specific approver, hybrid)
- View all company expenses
- Manage approval workflows

#### Manager
- Approve/reject expense submissions
- View team expenses
- Submit own expenses

#### Employee
- Submit expense claims with receipt upload
- Track expense status
- View approval history

## Tech Stack

### Frontend
- **React 18** - UI framework
- **Vite** - Build tool and dev server
- **TailwindCSS** - Utility-first CSS framework
- **React Query** - Data fetching and state management
- **React Router** - Client-side routing
- **Lucide React** - Icon library
- **Tesseract.js** - OCR processing

### Backend
- **Supabase** - Backend-as-a-Service
  - PostgreSQL database
  - Authentication
  - Storage
  - Row Level Security (RLS)
  - Edge Functions

### APIs
- **ExchangeRate API** - Currency conversion
- **REST Countries API** - Country/currency mapping

## Getting Started

### Prerequisites
- Node.js 18+ and npm/yarn
- Supabase account

### 1. Supabase Setup

1. Create a new Supabase project at [supabase.com](https://supabase.com)

2. Run the database schema:
   - Go to SQL Editor in Supabase Dashboard
   - Execute `infra/schema.sql`
   - Execute `infra/rls_policies.sql`

3. Create Storage Bucket:
   - Go to Storage in Supabase Dashboard
   - Create a new bucket named `receipts`
   - Set it to private (authenticated access only)

4. Get your credentials:
   - Go to Project Settings > API
   - Copy the `URL` and `anon/public` key

### 2. Local Setup

1. Clone the repository:
```bash
git clone <your-repo-url>
cd expenseura
```

2. Install dependencies:
```bash
npm install
```

3. Create environment file:
```bash
cp .env.example .env
```

4. Update `.env` with your Supabase credentials:
```env
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your-anon-key
VITE_EXCHANGE_API_KEY=optional
VITE_RESTCOUNTRIES_URL=https://restcountries.com/v3.1/all?fields=name,currencies
```

5. Start the development server:
```bash
npm run dev
```

6. Open [http://localhost:3000](http://localhost:3000)

## Usage Guide

### First Time Setup

1. **Sign Up** (Creates Admin Account)
   - Navigate to `/signup`
   - Enter your details
   - Choose company country (base currency auto-set)
   - First signup automatically creates company and admin user

2. **Create Approval Rule** (Admin)
   - Go to Approval Rules
   - Create at least one rule for expense approvals
   - Configure approvers, percentage, or specific approver

3. **Add Users** (Admin)
   - Go to Users
   - Add Managers and Employees
   - Assign manager relationships

### Employee Workflow

1. **Submit Expense**
   - Go to My Expenses
   - Click "New Expense"
   - Upload receipt (OCR auto-fills fields)
   - Review and correct OCR data
   - Select category, payment method
   - Submit for approval

2. **Track Status**
   - View expense list with status badges
   - Click to see approval history
   - View receipt and details

### Manager Workflow

1. **Review Approvals**
   - Go to Approvals
   - View pending expenses
   - Click to see full details and receipt
   - Approve or reject with comments

2. **View Team Expenses**
   - Go to All Expenses
   - Filter by status, category
   - Monitor team spending

### Admin Workflow

1. **Manage Users**
   - Create users with roles
   - Assign managers to employees
   - Update user information

2. **Configure Approval Rules**
   - Create rules with different types:
     - **Percentage**: Requires X% of approvers
     - **Specific**: Requires specific person (e.g., CFO)
     - **Hybrid**: Either specific OR percentage
   - Set sequential or parallel approval
   - Include manager approval requirement

## Database Schema

### Tables
- `companies` - Company information and base currency
- `profiles` - User profiles linked to Supabase auth
- `approval_rules` - Configurable approval workflows
- `expenses` - Expense records with amounts and status
- `expense_approvals` - Approval tracking and history

### Key Features
- Row Level Security (RLS) on all tables
- Foreign key constraints for data integrity
- Indexes for performance
- Stored procedure for approval processing

## API Integration

### Currency Conversion
Expenses are automatically converted to company base currency using real-time exchange rates:
```javascript
const amountBase = await convertCurrency(amount, fromCurrency, baseCurrency)
```

### OCR Processing
Receipt images are processed client-side:
```javascript
const parsed = await parseReceipt(file)
// Returns: { amount, date, merchant, rawText }
```

## Deployment

### Frontend (Vercel)

1. Push code to GitHub
2. Import project in Vercel
3. Add environment variables
4. Deploy

### Database (Supabase)
Already hosted - no additional deployment needed

## Project Structure

```
expenseura/
├── infra/
│   ├── schema.sql          # Database schema
│   ├── rls_policies.sql    # Security policies
│   └── seed.sql            # Sample data
├── src/
│   ├── components/
│   │   ├── Admin/          # Admin components
│   │   ├── Auth/           # Login/Signup
│   │   ├── Employee/       # Expense submission
│   │   ├── Manager/        # Approval components
│   │   └── Shared/         # Shared components
│   ├── hooks/              # Custom React hooks
│   │   ├── useAuth.js
│   │   ├── useExpenses.js
│   │   ├── useApprovals.js
│   │   └── useUsers.js
│   ├── lib/                # Utilities
│   │   ├── supabaseClient.js
│   │   ├── ocr.js
│   │   ├── currency.js
│   │   └── constants.js
│   ├── pages/              # Page components
│   ├── App.jsx             # Main app component
│   └── main.jsx            # Entry point
├── package.json
├── vite.config.js
├── tailwind.config.js
└── README.md
```

## Security

- **Row Level Security (RLS)**: All database tables protected
- **Authentication**: Supabase Auth with JWT tokens
- **Signed URLs**: Receipt access via temporary signed URLs
- **Role-Based Access**: Route and data access by user role
- **Input Validation**: Client and server-side validation

## Testing

### Unit Tests
```bash
npm test
```

### E2E Tests
```bash
npm run test:e2e
```

## Troubleshooting

### Common Issues

1. **OCR not working**
   - Ensure image quality is good
   - Try different image formats
   - Manual entry always available

2. **Currency conversion fails**
   - Check internet connection
   - API rate limits may apply
   - Falls back to original currency

3. **Upload fails**
   - Check file size (max 10MB)
   - Verify storage bucket permissions
   - Check Supabase storage quota

4. **RLS errors**
   - Verify policies are applied
   - Check user authentication
   - Review role assignments

## Contributing

1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

## License

MIT License - feel free to use for personal or commercial projects

## Support

For issues and questions:
- Create an issue on GitHub
- Check Supabase documentation
- Review the troubleshooting section

## Roadmap

- [ ] Email notifications for approvals
- [ ] Bulk expense upload
- [ ] Advanced reporting and analytics
- [ ] Mobile app (React Native)
- [ ] Integration with accounting software
- [ ] Mileage tracking
- [ ] Per diem calculations
- [ ] Budget management
- [ ] Expense policies and limits

## Acknowledgments

- Supabase for the amazing BaaS platform
- Tesseract.js for OCR capabilities
- TailwindCSS for the styling system
- React Query for data management
