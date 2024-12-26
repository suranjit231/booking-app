# Multi-Category Booking Application Schema Documentation

## Table of Contents
1. [Schema Overview](#schema-overview)
2. [Core Schemas](#core-schemas)
   - [User Schema](#user-schema)
   - [Business Schema](#business-schema)
   - [Service Schema](#service-schema)
   - [Staff Schema](#staff-schema)
   - [Slot Schema](#slot-schema)
   - [Booking Schema](#booking-schema)
   - [Category Schema](#category-schema)
   - [Review Schema](#review-schema)
3. [Schema Relationships](#schema-relationships)
4. [Common Scenarios](#common-scenarios)
5. [Edge Cases](#edge-cases)
6. [Best Practices](#best-practices)
7. [Implementation Notes](#implementation-notes)

## Schema Overview

The application uses MongoDB with Mongoose ODM for data modeling. Each schema is designed to handle various business scenarios while maintaining data integrity and scalability.

## Core Schemas

### User Schema
```javascript
const userSchema = new Schema({
  name: {
    type: String,
    required: true,
    trim: true,
    minlength: [2, 'Name must be at least 2 characters'],
    maxlength: [50, 'Name cannot exceed 50 characters']
  },
  email: {
    type: String,
    required: true,
    unique: true,
    lowercase: true,
    trim: true,
    validate: {
      validator: function(v) {
        return /^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/.test(v);
      },
      message: 'Please enter a valid email'
    }
  },
  phone: {
    type: String,
    required: true,
    validate: {
      validator: function(v) {
        return /^\+?[\d\s-]{10,}$/.test(v);
      },
      message: 'Please enter a valid phone number'
    }
  }
  // ... [Previous fields remain the same]
});

/*
USER SCHEMA SCENARIOS:

1. Registration Flows:
   - Standard email registration
   - Social media signup (OAuth)
   - Phone number verification
   - Business owner registration

2. Authentication Cases:
   - Password reset
   - Email verification
   - Two-factor authentication
   - Session management

3. Profile Management:
   - Profile completion tracking
   - Preference updates
   - Notification settings
   - Language/currency preferences

4. Booking Related:
   - Multiple bookings handling
   - Cancellation history
   - Favorite businesses
   - Review management

EDGE CASES:

1. Account Merging:
   - When user has multiple accounts
   - Social media account linking
   - Duplicate email prevention

2. Data Privacy:
   - GDPR compliance
   - Data deletion requests
   - Data export requests

3. Security:
   - Account lockout
   - Suspicious activity detection
   - Multiple device login

4. Verification:
   - Failed email verification
   - Bounced emails
   - Invalid phone numbers
*/
```

### Business Schema
```javascript
const businessSchema = new Schema({
  // ... [Previous fields remain the same]
  
  operationalDetails: {
    timeZone: {
      type: String,
      required: true,
      default: 'UTC'
    },
    maxBookingsPerDay: {
      type: Number,
      default: null // null means unlimited
    },
    maxAdvanceBookingDays: {
      type: Number,
      default: 30
    },
    instantBooking: {
      enabled: {
        type: Boolean,
        default: false
      },
      conditions: {
        minHoursBeforeBooking: {
          type: Number,
          default: 2
        },
        maxHoursBeforeBooking: {
          type: Number,
          default: 48
        }
      }
    }
  },
  
  analytics: {
    totalBookings: {
      type: Number,
      default: 0
    },
    completedBookings: {
      type: Number,
      default: 0
    },
    cancelledBookings: {
      type: Number,
      default: 0
    },
    averageRating: {
      type: Number,
      default: 0
    },
    revenue: {
      daily: [{
        date: Date,
        amount: Number
      }],
      monthly: [{
        month: Number,
        year: Number,
        amount: Number
      }]
    }
  }
});

/*
BUSINESS SCHEMA SCENARIOS:

1. Business Setup:
   - Initial registration
   - Category selection
   - Service configuration
   - Staff management
   - Working hours setup

2. Operational Management:
   - Holiday management
   - Special hours
   - Break time handling
   - Multiple location support

3. Service Delivery:
   - Booking confirmation flows
   - Cancellation handling
   - Staff assignment
   - Resource allocation

4. Financial Operations:
   - Payment processing
   - Refund handling
   - Commission calculation
   - Tax management

EDGE CASES:

1. Time Management:
   - Timezone conflicts
   - Daylight saving transitions
   - Holiday conflicts
   - Last-minute schedule changes

2. Capacity Issues:
   - Overbooking prevention
   - Staff availability conflicts
   - Resource double-booking
   - Walk-in handling

3. Service Disruptions:
   - Emergency closures
   - Staff no-shows
   - System downtime
   - Natural disasters

4. Business Changes:
   - Ownership transfer
   - Category changes
   - Location changes
   - Service modifications
*/
```

### Service Schema
```javascript
const serviceSchema = new Schema({
  // ... [Previous fields remain the same]
  
  scheduling: {
    concurrent: {
      type: Boolean,
      default: false
    },
    maxConcurrent: {
      type: Number,
      default: 1
    },
    preparation: {
      before: {
        type: Number,
        default: 0
      },
      after: {
        type: Number,
        default: 0
      }
    }
  },
  
  capacity: {
    minimum: {
      type: Number,
      default: 1
    },
    maximum: {
      type: Number,
      default: 1
    },
    groupBooking: {
      enabled: {
        type: Boolean,
        default: false
      },
      maxSize: {
        type: Number,
        default: 1
      },
      pricing: {
        type: String,
        enum: ['per_person', 'flat_rate', 'tiered'],
        default: 'per_person'
      }
    }
  }
});

/*
SERVICE SCHEMA SCENARIOS:

1. Service Configuration:
   - Basic service setup
   - Pricing models
   - Duration settings
   - Capacity planning

2. Availability Management:
   - Custom schedules
   - Staff assignment
   - Resource allocation
   - Seasonal variations

3. Booking Rules:
   - Advance booking
   - Cancellation policies
   - Payment requirements
   - Group booking rules

4. Special Handling:
   - Package services
   - Combined services
   - Custom durations
   - Variable pricing

EDGE CASES:

1. Timing Conflicts:
   - Service overlap
   - Staff availability
   - Resource conflicts
   - Break time handling

2. Pricing Complexity:
   - Dynamic pricing
   - Special offers
   - Package deals
   - Group discounts

3. Capacity Management:
   - Overbooking prevention
   - Resource allocation
   - Staff assignment
   - Walk-in handling

4. Service Modifications:
   - Mid-service changes
   - Duration adjustments
   - Price updates
   - Availability changes
*/
```

[Continue with similar detailed documentation for each schema...]

## Schema Relationships

### One-to-Many Relationships
1. User -> Bookings
   ```javascript
   // User Schema
   bookings: [{
     type: Schema.Types.ObjectId,
     ref: 'Booking'
   }]
   ```

2. Business -> Services
   ```javascript
   // Business Schema
   services: [{
     type: Schema.Types.ObjectId,
     ref: 'Service'
   }]
   ```

### Many-to-Many Relationships
1. Users <-> Businesses (Favorites)
   ```javascript
   // User Schema
   favoriteBusinesses: [{
     type: Schema.Types.ObjectId,
     ref: 'Business'
   }]
   ```

## Common Scenarios

### Booking Flow
1. User selects service
2. System checks availability
3. User selects slot
4. System reserves slot
5. User completes payment
6. System confirms booking

```javascript
// Example booking creation
const booking = new Booking({
  userId: user._id,
  businessId: business._id,
  serviceId: service._id,
  slotId: slot._id,
  status: 'pending',
  payment: {
    amount: service.price,
    status: 'pending'
  }
});
```

### Cancellation Flow
1. User requests cancellation
2. System checks cancellation policy
3. Calculate refund amount
4. Process cancellation
5. Update availability
6. Notify parties

```javascript
// Example cancellation handling
booking.status = 'cancelled';
booking.cancellation = {
  date: new Date(),
  reason: 'user_request',
  cancelledBy: 'user',
  refundStatus: 'pending'
};
```

## Edge Cases

### Concurrent Booking Attempts
```javascript
// Use transactions for slot reservation
const session = await mongoose.startSession();
session.startTransaction();
try {
  const slot = await Slot.findOne({ _id: slotId, status: 'available' });
  if (!slot) throw new Error('Slot no longer available');
  
  slot.status = 'booked';
  await slot.save();
  
  // Create booking
  await booking.save();
  
  await session.commitTransaction();
} catch (error) {
  await session.abortTransaction();
  throw error;
}
```

### Service Time Overlap
```javascript
// Check for overlapping bookings
const hasOverlap = await Booking.exists({
  staffId: staffId,
  date: bookingDate,
  $or: [
    {
      startTime: { $lt: endTime },
      endTime: { $gt: startTime }
    }
  ]
});
```

## Best Practices

### 1. Index Design
```javascript
// User Schema indexes
userSchema.index({ email: 1 }, { unique: true });
userSchema.index({ phone: 1 });
userSchema.index({ 'location.coordinates': '2dsphere' });

// Booking Schema indexes
bookingSchema.index({ userId: 1, status: 1 });
bookingSchema.index({ businessId: 1, date: 1 });
```

### 2. Data Validation
```javascript
// Custom validators
businessSchema.path('workingHours').validate(function(hours) {
  if (!hours || !hours.length) return false;
  return hours.every(hour => {
    return hour.shifts && hour.shifts.length > 0;
  });
}, 'Working hours must be properly configured');
```

### 3. Middleware Usage
```javascript
// Pre-save middleware for booking
bookingSchema.pre('save', async function(next) {
  if (this.isNew) {
    this.bookingNumber = await generateBookingNumber();
  }
  next();
});
```

## Implementation Notes

### 1. Performance Considerations
- Use appropriate indexes
- Implement caching for frequently accessed data
- Use aggregation pipelines for complex queries
- Implement pagination for large datasets

### 2. Security Measures
- Implement rate limiting
- Use field-level encryption for sensitive data
- Implement proper access control
- Regular security audits

### 3. Scalability Approaches
- Implement sharding strategies
- Use replica sets
- Implement caching layers
- Use message queues for async operations

### 4. Monitoring and Maintenance
- Set up logging
- Implement health checks
- Monitor database performance
- Regular backup strategies

## Conclusion

This comprehensive schema design covers the core functionality of a multi-category booking system while addressing various edge cases and scenarios. Regular review and updates are recommended as the system evolves and new requirements emerge.
