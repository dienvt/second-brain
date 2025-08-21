---
tags:
  - algorithm
  - leetcode
---
## Analyze
round-base proceduce -> for loop until have the result
1 right
ban ->
announce the victory 

[ X ] Có thể xếp toàn bộ senator vào một cái ring, xong loop through

Better is create two slice with earch Radient and Dive.
After that, just compare the index of those slice element
## Schedo

```go
dives []int
radiants []int
for len(dives) > 0 & len(radiants) > 0 {
	d, r := dives[0], radiants[0]
	dives, radiants = dives[1:], radiants[1:]
	if d < r {
		drives = append(dives, d + len(input))	
	} else {
		radiants = append(radiants, r + len(input))	
	}
}
return "radiant" if len(radiants)

```

