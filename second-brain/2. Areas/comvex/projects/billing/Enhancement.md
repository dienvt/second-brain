WTF:   
```

// Check if an AccountPricingPlan already exists for this account
existing, err := s.repo.FindByAccountId(ctx, data.AccountId)
if err != nil && !errors.IsNotFoundError(err) {
	panic(err)
}
```

