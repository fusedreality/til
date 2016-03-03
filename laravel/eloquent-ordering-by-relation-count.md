# Eloquent - Ordering by relation count

I've got a `belongsToMany` relation between two models: `Features` and `Users`
via a pivot table `feature_user`, and want to order my features by the number of users
attached to this relation. Turns out this isn't too easy!

The below works, but sadly doesn't allow you to paginate the results afterwards.
This is because by the time we've gotten to here, we've already retrieved the results
into a Collection.

```php
$data = Feature::with('users')
                ->get()
                ->sortBy(function ($feature) {
                    return $feature->users->count();
                });
```

It seems the only way to do this, with pagination in mind from the start, is to
join with the relation manually:

```php
$data = Feature::leftJoin('feature_user', 'features.id', '=', 'features.feature_id')
               ->groupBy('features.id')
               ->orderBy('feature_user_count')
               ->paginate(
                   null,
                   [
                       'features.*',
                       \DB::raw('COUNT(`' . \DB::getTablePrefix() . 'feature_user`.`user_id`) AS `feature_user_count`')
                   ]
               );
```
