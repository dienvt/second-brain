# Shared business-type & usage-metadata helpers — Implementation Plan  
  
> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.  
  
**Goal:** Remove the ~12 duplicated business-type helpers and the duplicated usage-metadata marshaller from `sms_reply` and `contact_nurturing` by extracting them into two new shared packages.  
  
**Architecture:** A `businesstype.Selector` struct (holds the digima adapter, injected via the app provider) exposes a single public `Resolve` method; all helper logic is package-private. A `usagemetadata.Marshal` pure function replaces the duplicated marshaller. Both services consume the new packages; the old copies and their TODOs are deleted.  
  
**Tech Stack:** Go, testify (`require`/`mock`), Mockery v3 mock `digimaAdapter.MockAdapter`, GORM-backed services wired via `backend-service-go-framework` DI container.  
  
**Test/build commands (run inside the `agent` Docker container):**  
- Build: `docker exec agent go build -buildvcs=false -o ./tmp/main .`  
- Test a package: `docker exec agent go test -p 1 --count=1 ./domain/services/businesstype/`  
- Full suite: `docker exec agent go test -p 1 --count=1 ./...`  
- Format: `docker exec agent gci write --skip-generated --custom-order -s standard -s alias -s default <files>`  
  
---  
  
## File Structure  

**New:**  
- `domain/services/usagemetadata/usagemetadata.go` — `Marshal` + unexported `aggregated` struct.  
- `domain/services/usagemetadata/usagemetadata_test.go` — unit tests.  
- `domain/services/businesstype/selector.go` — `Selector`, `NewSelector`, `Resolve`, + client-dependent unexported methods.  
- `domain/services/businesstype/helpers.go` — pure unexported helpers.  
- `domain/services/businesstype/selector_test.go` — consolidated unit tests.  
  
**Modified:**  
- `domain/services/sms_reply/service.go` — remove `marshalUsageMetadata`/`aggregatedUsageMetadata`? (these live in `pipeline.go` for sms_reply — see Task 2), add `BusinessTypeSelector` field, call `usagemetadata.Marshal`.  
- `domain/services/sms_reply/pipeline.go` — delete usage-metadata copy; call `usagemetadata.Marshal`; `tracedGetBusinessTypeInfo` calls `s.BusinessTypeSelector.Resolve`.  
- `domain/services/sms_reply/business_type.go` — delete moved helpers + `getBusinessTypeInfo` + TODO; keep `resolveBusinessType` & constants.  
- `domain/services/sms_reply/business_type_test.go` — delete moved tests; keep `resolveBusinessType` tests.  
- `domain/services/contact_nurturing/service.go` — delete usage-metadata copy + helpers + `getBusinessTypeInfo` + both TODOs; add `BusinessTypeSelector` field; call `usagemetadata.Marshal`; `tracedGetBusinessTypeInfo` calls `s.BusinessTypeSelector.Resolve`.  
- `domain/services/contact_nurturing/business_type_test.go` — delete (all tests move to shared pkg).  
- `domain/services/contact_nurturing/service_test.go` — delete the moved business-type tests; inject `BusinessTypeSelector` where the resolution path runs.  
- `domain/services/sms_reply/service_test.go` — inject `BusinessTypeSelector` where the resolution path runs.  
- `app/providers/app.go` — set `BusinessTypeSelector` in both registrations.  
  
---  
  
## Task 1: Create the `usagemetadata` package (test-first)  
  
**Files:**  
- Create: `domain/services/usagemetadata/usagemetadata_test.go`  
- Create: `domain/services/usagemetadata/usagemetadata.go`  
  
- [ ] **Step 1: Write the failing test**  
  
`domain/services/usagemetadata/usagemetadata_test.go`:  
  
```go  
// Package usagemetadata hosts the usagemetadata package.  
package usagemetadata  
  
import (  
    "testing"  
    agentAdapter "github.com/comvex-jp/digima-backend-agent/domain/adapters/agent"  
    "github.com/stretchr/testify/require")  
  
func TestMarshalReturnsNilForEmptyMap(t *testing.T) {  
    t.Parallel()  
    require.Nil(t, Marshal(map[string]agentAdapter.TokenUsage{}))}  
  
func TestMarshalReturnsNilWhenAllCountsZero(t *testing.T) {  
    t.Parallel()  
    require.Nil(t, Marshal(map[string]agentAdapter.TokenUsage{       "action_selector": {},    }))}  
  
func TestMarshalSumsCountsAcrossAgents(t *testing.T) {  
    t.Parallel()  
    result := Marshal(map[string]agentAdapter.TokenUsage{       "a": {PromptTokenCount: 1, CandidatesTokenCount: 2, ThoughtsTokenCount: 3, TotalTokenCount: 6},       "b": {PromptTokenCount: 10, CandidatesTokenCount: 20, ThoughtsTokenCount: 30, TotalTokenCount: 60},    })  
    require.NotNil(t, result)    require.JSONEq(t, `{"prompt_token_count":11,"candidates_token_count":22,"thoughts_token_count":33,"total_token_count":66}`, *result)}  
```  
  
- [ ] **Step 2: Run test to verify it fails (compile error — Marshal undefined)**  
  
Run: `docker exec agent go test -p 1 --count=1 ./domain/services/usagemetadata/`  
Expected: FAIL — `undefined: Marshal`.  
  
- [ ] **Step 3: Write minimal implementation**  
  
`domain/services/usagemetadata/usagemetadata.go`:  
  
```go  
// Package usagemetadata aggregates per-agent token usage into the flat JSON  
// shape consumed by the Slack message-evaluator workflow's usage_metadata field.  
package usagemetadata  
  
import (  
    "encoding/json"  
    agentAdapter "github.com/comvex-jp/digima-backend-agent/domain/adapters/agent")  
  
// aggregated mirrors the Python UsageMetadata pydantic model  
// (digima-backend-ai-agent/domain/adapters/slack/dto.py:16) — a flat sum across  
// all agents in the run. Field names match the Python schema so the MCP-side  
// Slack workflow validator accepts the payload.  
type aggregated struct {  
    PromptTokenCount     int32 `json:"prompt_token_count"`    CandidatesTokenCount int32 `json:"candidates_token_count"`    ThoughtsTokenCount   int32 `json:"thoughts_token_count"`    TotalTokenCount      int32 `json:"total_token_count"`}  
  
// Marshal sums per-agent token counts and serializes the result. Returns nil  
// when the aggregated sum is zero so the field is omitted from the wire payload  
// (matches Python behavior of skipping the field when no LLM call ran).  
func Marshal(usages map[string]agentAdapter.TokenUsage) *string {  
    sum := aggregated{}    for _, usage := range usages {       sum.PromptTokenCount += usage.PromptTokenCount       sum.CandidatesTokenCount += usage.CandidatesTokenCount       sum.ThoughtsTokenCount += usage.ThoughtsTokenCount       sum.TotalTokenCount += usage.TotalTokenCount    }  
    if sum == (aggregated{}) {       return nil    }  
    payload, err := json.Marshal(sum)    if err != nil {       return nil    }  
    encoded := string(payload)  
    return &encoded}  
```  
  
- [ ] **Step 4: Run test to verify it passes**  
  
Run: `docker exec agent go test -p 1 --count=1 ./domain/services/usagemetadata/`  
Expected: PASS (3 tests).  
  
- [ ] **Step 5: Commit**  
  
```bash  
git add domain/services/usagemetadata/git commit -m "feat: add shared usagemetadata.Marshal package [DGM2-30013]"```  
  
---  
  
## Task 2: Replace duplicated usage-metadata in both services  
  
**Files:**  
- Modify: `domain/services/sms_reply/pipeline.go` (delete struct+func at lines 170-206; 3 call sites: `service.go:375`, `service.go:406`, `pipeline.go:268`)  
- Modify: `domain/services/contact_nurturing/service.go` (delete struct+func at lines 360-402; comment block 360-368; 2 call sites: lines 290, 310)  
  
- [ ] **Step 1: sms_reply — delete the duplicated definition**  
  
In `domain/services/sms_reply/pipeline.go`, delete the entire block from the `// aggregatedUsageMetadata mirrors ...` comment (line 170) through the end of `marshalUsageMetadata` (line 206), i.e. both the `aggregatedUsageMetadata` struct and the `marshalUsageMetadata` func.  
  
- [ ] **Step 2: sms_reply — update the 3 call sites**  
  
Replace `marshalUsageMetadata(` with `usagemetadata.Marshal(` at:  
- `domain/services/sms_reply/service.go:375`  
- `domain/services/sms_reply/service.go:406`  
- `domain/services/sms_reply/pipeline.go:268`  
  
Add the import `"github.com/comvex-jp/digima-backend-agent/domain/services/usagemetadata"` (non-aliased group) to **both** `service.go` and `pipeline.go`. If `encoding/json` becomes unused in `pipeline.go` after deletion, remove it (run build to confirm).  
  
- [ ] **Step 3: contact_nurturing — delete the duplicated definition**  
  
In `domain/services/contact_nurturing/service.go`, delete the block from the `// aggregatedUsageMetadata mirrors ...` comment (line 360) through the end of `marshalUsageMetadata` (line 402). This also removes the `// TODO: [Cleanup] Lift this and marshalUsageMetadata into a shared package` comment (lines 365-368).  
  
- [ ] **Step 4: contact_nurturing — update the 2 call sites**  
  
Replace `marshalUsageMetadata(` with `usagemetadata.Marshal(` at `domain/services/contact_nurturing/service.go:290` and `:310`. Add the `usagemetadata` import. If `encoding/json` becomes unused in `service.go`, remove it (run build to confirm).  
  
- [ ] **Step 5: Build + format + run both suites**  
  
```bash  
docker exec agent go build -buildvcs=false -o ./tmp/main .docker exec agent gci write --skip-generated --custom-order -s standard -s alias -s default \  domain/services/sms_reply/service.go domain/services/sms_reply/pipeline.go \  domain/services/contact_nurturing/service.godocker exec agent go test -p 1 --count=1 ./domain/services/sms_reply/ ./domain/services/contact_nurturing/```  
Expected: build OK, tests PASS (behavior preserved — sms_reply's all-zero non-empty edge is the only semantic change, not covered by existing tests).  
  
- [ ] **Step 6: Commit**  
  
```bash  
git add domain/services/sms_reply/ domain/services/contact_nurturing/git commit -m "refactor: use shared usagemetadata.Marshal, drop duplicates [DGM2-30013]"```  
  
---  
  
## Task 3: Create the `businesstype` package (test-first)  
  
**Files:**  
- Create: `domain/services/businesstype/selector_test.go`  
- Create: `domain/services/businesstype/selector.go`  
- Create: `domain/services/businesstype/helpers.go`  
  
- [ ] **Step 1: Write the failing/consolidated test**  
  
`domain/services/businesstype/selector_test.go` — this consolidates the tests currently in both `business_type_test.go` files (use the `Selector` instead of `Service`). Note `new(uint64(N))` is intentional (matches existing code):  
  
```go  
// Package businesstype hosts the businesstype package.  
package businesstype  
  
import (  
    "context"    "errors"    "testing"  
    digimaAdapter "github.com/comvex-jp/digima-backend-agent/domain/adapters/digima"    agentsettingmodel "github.com/comvex-jp/digima-backend-agent/domain/models/agent_setting"    businesstypesetting "github.com/comvex-jp/digima-backend-agent/domain/models/business_type_setting"  
    "github.com/stretchr/testify/mock"    "github.com/stretchr/testify/require")  
  
func TestResolveReturnsEmptyWhenAgentSettingNil(t *testing.T) {  
    t.Parallel()  
    businessType, companyStrength := NewSelector(nil).Resolve(context.Background(), 1, 2, nil)  
    require.Equal(t, "", businessType)    require.Equal(t, "", companyStrength)}  
  
func TestResolveReturnsEmptyWhenNoBusinessTypeSettings(t *testing.T) {  
    t.Parallel()  
    businessType, companyStrength := NewSelector(nil).Resolve(context.Background(), 1, 2, &agentsettingmodel.Model{})  
    require.Equal(t, "", businessType)    require.Equal(t, "", companyStrength)}  
  
func TestResolveReturnsSingleSettingWithoutCallingDigima(t *testing.T) {  
    t.Parallel()  
    setting := &agentsettingmodel.Model{       BusinessTypeSettings: []businesstypesetting.Model{          {BusinessType: businesstypesetting.BusinessTypeCustomBuiltHomes, CompanyStrength: "alpha"},       },    }  
    selector := NewSelector(digimaAdapter.NewMockAdapter(t))    businessType, companyStrength := selector.Resolve(context.Background(), 1, 2, setting)  
    require.Equal(t, "custom_built_homes", businessType)    require.Equal(t, "alpha", companyStrength)}  
  
func TestResolveResolvesDynamicGroupMembership(t *testing.T) {  
    t.Parallel()  
    setting := &agentsettingmodel.Model{       BusinessTypeSettings: []businesstypesetting.Model{          {BusinessType: businesstypesetting.BusinessTypeRealEstatePurchase, IsPrimary: true, CompanyStrength: "primary", ContactGroupIds: []uint64{100}},          {BusinessType: businesstypesetting.BusinessTypeRemodeling, CompanyStrength: "beta", ContactGroupIds: []uint64{200}},       },    }  
    digimaClient := digimaAdapter.NewMockAdapter(t)    digimaClient.EXPECT().GetContactGroupMemberships(mock.Anything, uint64(1), uint64(42), []uint64{100, 200}).Return([]digimaAdapter.Group{       {Id: 100, Name: "real_estate_purchase", Type: "dynamic", HasContact: false},       {Id: 200, Name: "remodeling", Type: "dynamic", HasContact: true},    }, nil)  
    businessType, companyStrength := NewSelector(digimaClient).Resolve(context.Background(), 1, 42, setting)  
    require.Equal(t, "remodeling", businessType)    require.Equal(t, "beta", companyStrength)}  
  
func TestResolveFallsBackToFirstWhenNoMatch(t *testing.T) {  
    t.Parallel()  
    setting := &agentsettingmodel.Model{       BusinessTypeSettings: []businesstypesetting.Model{          {BusinessType: businesstypesetting.BusinessTypeCustomBuiltHomes, CompanyStrength: "alpha", ContactGroupIds: []uint64{100}},          {BusinessType: businesstypesetting.BusinessTypeRemodeling, CompanyStrength: "beta", ContactGroupIds: []uint64{200}},       },    }  
    digimaClient := digimaAdapter.NewMockAdapter(t)    digimaClient.EXPECT().GetContactGroupMemberships(mock.Anything, uint64(1), uint64(42), []uint64{100, 200}).Return([]digimaAdapter.Group{       {Id: 100, Name: "custom_built_homes", Type: "dynamic", HasContact: false},       {Id: 200, Name: "remodeling", Type: "dynamic", HasContact: false},    }, nil)  
    businessType, companyStrength := NewSelector(digimaClient).Resolve(context.Background(), 1, 42, setting)  
    require.Equal(t, "custom_built_homes", businessType)    require.Equal(t, "alpha", companyStrength)}  
  
func TestResolveFallsBackOnDigimaError(t *testing.T) {  
    t.Parallel()  
    setting := &agentsettingmodel.Model{       BusinessTypeSettings: []businesstypesetting.Model{          {BusinessType: businesstypesetting.BusinessTypeCustomBuiltHomes, IsPrimary: true, CompanyStrength: "alpha", ContactGroupIds: []uint64{100}},          {BusinessType: businesstypesetting.BusinessTypeRemodeling, CompanyStrength: "beta", ContactGroupIds: []uint64{200}},       },    }  
    digimaClient := digimaAdapter.NewMockAdapter(t)    digimaClient.EXPECT().GetContactGroupMemberships(mock.Anything, uint64(1), uint64(42), []uint64{100, 200}).Return(nil, errors.New("boom"))  
    businessType, companyStrength := NewSelector(digimaClient).Resolve(context.Background(), 1, 42, setting)  
    require.Equal(t, "custom_built_homes", businessType)    require.Equal(t, "alpha", companyStrength)}  
  
func TestResolveReturnsEmptyWhenAllGatedOut(t *testing.T) {  
    t.Parallel()  
    agentSetting := &agentsettingmodel.Model{       BusinessTypeSettings: []businesstypesetting.Model{          {BusinessType: businesstypesetting.BusinessTypeRemodeling, TargetContactGroupId: new(uint64(20))},       },    }  
    digimaClient := digimaAdapter.NewMockAdapter(t)    digimaClient.EXPECT().GetContactGroupMemberships(mock.Anything, uint64(1), uint64(2), []uint64{20}).Return([]digimaAdapter.Group{       {Id: 20, HasContact: false},    }, nil)  
    businessType, companyStrength := NewSelector(digimaClient).Resolve(context.Background(), 1, 2, agentSetting)  
    require.Empty(t, businessType)    require.Empty(t, companyStrength)}  
  
func TestSelectBusinessTypeSettingReturnsPrimaryWhenNoContactGroupMatch(t *testing.T) {  
    t.Parallel()  
    settings := []businesstypesetting.Model{       {BusinessType: businesstypesetting.BusinessTypeCustomBuiltHomes, ContactGroupIds: []uint64{100}},       {BusinessType: businesstypesetting.BusinessTypeRemodeling, IsPrimary: true, ContactGroupIds: []uint64{200}},    }  
    selected := selectBusinessTypeSetting(nil, settings)  
    require.NotNil(t, selected)    require.Equal(t, businesstypesetting.BusinessTypeRemodeling, selected.BusinessType)}  
  
func TestSelectBusinessTypeSettingPicksMatchingGroup(t *testing.T) {  
    t.Parallel()  
    settings := []businesstypesetting.Model{       {BusinessType: businesstypesetting.BusinessTypeCustomBuiltHomes, ContactGroupIds: []uint64{100}},       {BusinessType: businesstypesetting.BusinessTypeRemodeling, ContactGroupIds: []uint64{200}},    }  
    selected := selectBusinessTypeSetting([]uint64{200}, settings)  
    require.NotNil(t, selected)    require.Equal(t, businesstypesetting.BusinessTypeRemodeling, selected.BusinessType)}  
  
func TestSelectBusinessTypeSettingReturnsNilWhenSettingsEmpty(t *testing.T) {  
    t.Parallel()  
    require.Nil(t, selectBusinessTypeSetting(nil, nil))}  
  
func TestFilterByTargetContactGroup_NoTriggerGroupsPassesAll(t *testing.T) {  
    t.Parallel()  
    settings := []businesstypesetting.Model{       {BusinessType: businesstypesetting.BusinessTypeCustomBuiltHomes},       {BusinessType: businesstypesetting.BusinessTypeRemodeling},    }  
    selector := NewSelector(digimaAdapter.NewMockAdapter(t))    filtered := selector.filterByTargetContactGroup(context.Background(), 1, 2, settings)  
    require.Len(t, filtered, 2)}  
  
func TestFilterByTargetContactGroup_KeepsOnlyMemberGroup(t *testing.T) {  
    t.Parallel()  
    settings := []businesstypesetting.Model{       {BusinessType: businesstypesetting.BusinessTypeCustomBuiltHomes, TargetContactGroupId: new(uint64(10))},       {BusinessType: businesstypesetting.BusinessTypeRemodeling, TargetContactGroupId: new(uint64(20))},    }  
    digimaClient := digimaAdapter.NewMockAdapter(t)    digimaClient.EXPECT().GetContactGroupMemberships(mock.Anything, uint64(1), uint64(2), []uint64{10, 20}).Return([]digimaAdapter.Group{       {Id: 10, HasContact: false},       {Id: 20, HasContact: true},    }, nil)  
    filtered := NewSelector(digimaClient).filterByTargetContactGroup(context.Background(), 1, 2, settings)  
    require.Len(t, filtered, 1)    require.Equal(t, businesstypesetting.BusinessTypeRemodeling, filtered[0].BusinessType)}  
  
func TestFilterByTargetContactGroup_DropsAllWhenMemberOfNone(t *testing.T) {  
    t.Parallel()  
    settings := []businesstypesetting.Model{       {BusinessType: businesstypesetting.BusinessTypeCustomBuiltHomes, TargetContactGroupId: new(uint64(10))},       {BusinessType: businesstypesetting.BusinessTypeRemodeling, TargetContactGroupId: new(uint64(20))},    }  
    digimaClient := digimaAdapter.NewMockAdapter(t)    digimaClient.EXPECT().GetContactGroupMemberships(mock.Anything, uint64(1), uint64(2), []uint64{10, 20}).Return([]digimaAdapter.Group{       {Id: 10, HasContact: false},       {Id: 20, HasContact: false},    }, nil)  
    filtered := NewSelector(digimaClient).filterByTargetContactGroup(context.Background(), 1, 2, settings)  
    require.Empty(t, filtered)}  
  
func TestFilterByTargetContactGroup_NilAlwaysSurvivesWithMember(t *testing.T) {  
    t.Parallel()  
    settings := []businesstypesetting.Model{       {BusinessType: businesstypesetting.BusinessTypeRealEstatePurchase},       {BusinessType: businesstypesetting.BusinessTypeCustomBuiltHomes, TargetContactGroupId: new(uint64(10))},       {BusinessType: businesstypesetting.BusinessTypeRemodeling, TargetContactGroupId: new(uint64(20))},    }  
    digimaClient := digimaAdapter.NewMockAdapter(t)    digimaClient.EXPECT().GetContactGroupMemberships(mock.Anything, uint64(1), uint64(2), []uint64{10, 20}).Return([]digimaAdapter.Group{       {Id: 10, HasContact: true},       {Id: 20, HasContact: false},    }, nil)  
    filtered := NewSelector(digimaClient).filterByTargetContactGroup(context.Background(), 1, 2, settings)  
    require.Len(t, filtered, 2)    require.Equal(t, businesstypesetting.BusinessTypeRealEstatePurchase, filtered[0].BusinessType)    require.Equal(t, businesstypesetting.BusinessTypeCustomBuiltHomes, filtered[1].BusinessType)}  
```  
  
- [ ] **Step 2: Run test to verify it fails (compile error)**  
  
Run: `docker exec agent go test -p 1 --count=1 ./domain/services/businesstype/`  
Expected: FAIL — `undefined: NewSelector`, `undefined: selectBusinessTypeSetting`.  
  
- [ ] **Step 3: Write `selector.go` (client-dependent code)**  
  
`domain/services/businesstype/selector.go`:  
  
```go  
// Package businesstype resolves the business type and company strength for a  
// contact from an agent setting's business-type settings.  
package businesstype  
  
import (  
    "context"  
    digimaAdapter "github.com/comvex-jp/digima-backend-agent/domain/adapters/digima"    agentsettingmodel "github.com/comvex-jp/digima-backend-agent/domain/models/agent_setting"    businesstypesetting "github.com/comvex-jp/digima-backend-agent/domain/models/business_type_setting"  
    "github.com/comvex-jp/backend-service-go-framework/v8/logger")  
  
// Selector resolves business-type information for a contact. It depends only on  
// the digima adapter, never on a service struct.  
type Selector struct {  
    DigimaClient digimaAdapter.Adapter}  
  
// NewSelector builds a Selector backed by the given digima client.  
func NewSelector(client digimaAdapter.Adapter) Selector {  
    return Selector{DigimaClient: client}}  
  
// Resolve returns the (businessType, companyStrength) for the contact based on  
// the agent setting's business-type settings and the contact's group  
// memberships. Returns empty strings when no setting applies.  
func (s Selector) Resolve(ctx context.Context, accountId, contactId uint64, agentSetting *agentsettingmodel.Model) (string, string) {  
    if agentSetting == nil {       return "", ""    }  
    settings := s.filterByTargetContactGroup(ctx, accountId, contactId, agentSetting.BusinessTypeSettings)    if len(settings) == 0 {       return "", ""    }  
    if len(settings) == 1 {       return settings[0].BusinessType.String(), settings[0].CompanyStrength    }  
    contactGroupIds := s.resolveBusinessTypeContactGroupIds(ctx, accountId, contactId, settings)  
    setting := selectBusinessTypeSetting(contactGroupIds, settings)    if setting != nil {       return setting.BusinessType.String(), setting.CompanyStrength    }  
    defaultSetting := settings[0]  
    return defaultSetting.BusinessType.String(), defaultSetting.CompanyStrength}  
  
func (s Selector) filterByTargetContactGroup(ctx context.Context, accountId, contactId uint64, settings []businesstypesetting.Model) []businesstypesetting.Model {  
    targetGroupIds := collectTargetContactGroupIds(settings)    if len(targetGroupIds) == 0 {       return settings    }  
    memberGroupIds := s.resolveContactGroupMemberships(ctx, accountId, contactId, targetGroupIds)  
    filtered := make([]businesstypesetting.Model, 0, len(settings))    for _, setting := range settings {       if isAllowedByTargetContactGroup(setting, memberGroupIds) {          filtered = append(filtered, setting)       }    }  
    return filtered}  
  
func (s Selector) resolveContactGroupMemberships(ctx context.Context, accountId, contactId uint64, groupIds []uint64) map[uint64]struct{} {  
    memberGroupIds := map[uint64]struct{}{}  
    groups, err := s.DigimaClient.GetContactGroupMemberships(ctx, accountId, contactId, groupIds)    if err != nil {       logger.Logw(ctx, logger.WarnLevel, "domain.services.businesstype.resolveContactGroupMemberships.failed", "accountId", accountId, "contactId", contactId, "error", err)  
       return memberGroupIds    }  
    for _, g := range groups {       if g.HasContact {          memberGroupIds[g.Id] = struct{}{}       }    }  
    return memberGroupIds}  
  
func (s Selector) resolveBusinessTypeContactGroupIds(ctx context.Context, accountId, contactId uint64, settings []businesstypesetting.Model) []uint64 {  
    candidateGroupIds := collectBusinessTypeGroupIds(settings)    if len(candidateGroupIds) == 0 {       return nil    }  
    groups, err := s.DigimaClient.GetContactGroupMemberships(ctx, accountId, contactId, candidateGroupIds)    if err != nil {       logger.Logw(ctx, logger.WarnLevel, "domain.services.businesstype.resolveBusinessTypeContactGroupIds.failed", "accountId", accountId, "contactId", contactId, "error", err)  
       return nil    }  
    memberships := make([]uint64, 0, len(groups))    for _, g := range groups {       if !g.HasContact {          continue       }  
       memberships = append(memberships, g.Id)    }  
    return memberships}  
```  
  
- [ ] **Step 4: Write `helpers.go` (pure functions)**  
  
`domain/services/businesstype/helpers.go`:  
  
```go  
package businesstype  
  
import (  
    "math/rand"  
    businesstypesetting "github.com/comvex-jp/digima-backend-agent/domain/models/business_type_setting")  
  
func collectTargetContactGroupIds(settings []businesstypesetting.Model) []uint64 {  
    seen := map[uint64]struct{}{}  
    var result []uint64  
    for _, setting := range settings {       if setting.TargetContactGroupId == nil {          continue       }  
       groupId := *setting.TargetContactGroupId       if _, ok := seen[groupId]; ok {          continue       }  
       seen[groupId] = struct{}{}       result = append(result, groupId)    }  
    return result}  
  
func isAllowedByTargetContactGroup(setting businesstypesetting.Model, memberGroupIds map[uint64]struct{}) bool {  
    if setting.TargetContactGroupId == nil {       return true    }  
    _, ok := memberGroupIds[*setting.TargetContactGroupId]  
    return ok}  
  
func selectBusinessTypeSetting(contactGroupIds []uint64, settings []businesstypesetting.Model) *businesstypesetting.Model {  
    if len(settings) == 0 {       return nil    }  
    if len(settings) == 1 {       return &settings[0]    }  
    if len(contactGroupIds) == 1 {       return findBusinessTypeSettingByGroupId(settings, contactGroupIds[0])    }  
    primary := findPrimaryBusinessTypeSetting(settings)    if primary != nil && (len(contactGroupIds) == 0 || hasIntersectingGroupId(primary.ContactGroupIds, contactGroupIds)) {       return primary    }  
    var matched []*businesstypesetting.Model  
    for _, contactGroupId := range contactGroupIds {       setting := findBusinessTypeSettingByGroupId(settings, contactGroupId)       if setting != nil {          matched = append(matched, setting)       }    }  
    if len(matched) == 0 {       return primary    }  
    return matched[rand.Intn(len(matched))]}  
  
func collectBusinessTypeGroupIds(settings []businesstypesetting.Model) []uint64 {  
    seen := map[uint64]struct{}{}  
    var result []uint64  
    for _, setting := range settings {       for _, groupId := range setting.ContactGroupIds {          if _, ok := seen[groupId]; ok {             continue          }  
          seen[groupId] = struct{}{}          result = append(result, groupId)       }    }  
    return result}  
  
func findPrimaryBusinessTypeSetting(settings []businesstypesetting.Model) *businesstypesetting.Model {  
    for i := range settings {       if !settings[i].IsPrimary {          continue       }  
       return &settings[i]    }  
    return nil}  
  
func findBusinessTypeSettingByGroupId(settings []businesstypesetting.Model, contactGroupId uint64) *businesstypesetting.Model {  
    for i := range settings {       if !hasGroupId(settings[i].ContactGroupIds, contactGroupId) {          continue       }  
       return &settings[i]    }  
    return nil}  
  
func hasIntersectingGroupId(sourceGroupIds, targetGroupIds []uint64) bool {  
    for _, sourceGroupId := range sourceGroupIds {       if !hasGroupId(targetGroupIds, sourceGroupId) {          continue       }  
       return true    }  
    return false}  
  
func hasGroupId(groupIds []uint64, targetGroupId uint64) bool {  
    for _, groupId := range groupIds {       if groupId != targetGroupId {          continue       }  
       return true    }  
    return false}  
```  
  
- [ ] **Step 5: Run test to verify it passes**  
  
Run: `docker exec agent go test -p 1 --count=1 ./domain/services/businesstype/`  
Expected: PASS (all ~16 tests).  
  
- [ ] **Step 6: Format + commit**  
  
```bash  
docker exec agent gci write --skip-generated --custom-order -s standard -s alias -s default \  domain/services/businesstype/selector.go domain/services/businesstype/helpers.go domain/services/businesstype/selector_test.gogit add domain/services/businesstype/git commit -m "feat: add shared businesstype.Selector package [DGM2-30013]"```  
  
---  
  
## Task 4: Wire `BusinessTypeSelector` into both services and remove duplicates  
  
**Files:**  
- Modify: `domain/services/sms_reply/service.go:57-58` (struct field)  
- Modify: `domain/services/sms_reply/pipeline.go:45` (call site)  
- Modify: `domain/services/sms_reply/business_type.go` (delete moved code)  
- Modify: `domain/services/contact_nurturing/service.go` (struct field, call site, delete moved code)  
- Modify: `app/providers/app.go:56-72` and `:128-143`  
  
- [ ] **Step 1: Add the field to both Service structs**  
  
In `domain/services/sms_reply/service.go`, inside `type Service struct {` (after line 58 `DigimaClient digimaAdapter.Adapter`), add:  
  
```go  
    BusinessTypeSelector     businesstype.Selector  
```  
  
Add import (non-aliased group): `"github.com/comvex-jp/digima-backend-agent/domain/services/businesstype"`.  
  
In `domain/services/contact_nurturing/service.go`, inside `type Service struct {` (after line 48), add:  
  
```go  
    BusinessTypeSelector    businesstype.Selector  
```  
  
Add the same import.  
  
- [ ] **Step 2: Update both `tracedGetBusinessTypeInfo` call sites**  
  
In `domain/services/sms_reply/pipeline.go:45` change:  
  
```go  
       bt, cs := s.getBusinessTypeInfo(ctx, accountId, contactId, agentSetting)  
```  
to:  
```go  
       bt, cs := s.BusinessTypeSelector.Resolve(ctx, accountId, contactId, agentSetting)  
```  
  
Make the identical change in `domain/services/contact_nurturing/pipeline.go` (the `s.getBusinessTypeInfo(...)` line inside `tracedGetBusinessTypeInfo`).  
  
- [ ] **Step 3: Delete moved helpers from sms_reply**  
  
In `domain/services/sms_reply/business_type.go`, delete:  
- The `// TODO: extract shared business-type selection helper ...` comment (line 41) and `getBusinessTypeInfo` (lines 42-66).  
- `filterByTargetContactGroup` (68-84), `resolveContactGroupMemberships` (86-103), `collectTargetContactGroupIds` (105-125), `isAllowedByTargetContactGroup` (127-135), `resolveBusinessTypeContactGroupIds` (137-160), `selectBusinessTypeSetting` (162-194), `collectBusinessTypeGroupIds` (196-213), `findPrimaryBusinessTypeSetting` (215-225), `findBusinessTypeSettingByGroupId` (227-237), `hasIntersectingGroupId` (239-249), `hasGroupId` (251-261).  
  
**Keep** lines 1-39: the package comment, `defaultBusinessType`, `bracketedPrefixPattern`, and `resolveBusinessType`. After deletion the file's imports likely reduce to `regexp` and `strings` only — remove now-unused `context`, `math/rand`, `agentsettingmodel`, `businesstypesetting`, `logger` (run build to confirm exactly which).  
  
- [ ] **Step 4: Delete moved helpers from contact_nurturing**  
  
In `domain/services/contact_nurturing/service.go`, delete `getBusinessTypeInfo` (465-489), `filterByTargetContactGroup` (491-507), `resolveContactGroupMemberships` (509-526), `collectTargetContactGroupIds` (528-548), `isAllowedByTargetContactGroup` (550-558), `resolveBusinessTypeContactGroupIds` (560-583), `selectBusinessTypeSetting` (608-640), `collectBusinessTypeGroupIds` (642-659), `findPrimaryBusinessTypeSetting` (661-671), `findBusinessTypeSettingByGroupId` (673-683), `hasIntersectingGroupId` (685-695), `hasGroupId` (697-704+).  
  
**Keep** `getContactStatuses` (585-606) — it is not a business-type helper. `math/rand` may become unused after this — remove if build complains.  
  
- [ ] **Step 5: Wire the provider (both registrations)**  
  
In `app/providers/app.go`, `registerSmsReplyService` (struct literal at line 56), add a field right after `DigimaClient:`:  
  
```go  
       BusinessTypeSelector:     businesstype.NewSelector(a.Resolve(digimaAdapter.AdapterName).(digimaAdapter.Adapter)),  
```  
  
In `registerContactNurturingService` (struct literal at line 128), add:  
  
```go  
       BusinessTypeSelector:    businesstype.NewSelector(a.Resolve(digimaAdapter.AdapterName).(digimaAdapter.Adapter)),  
```  
  
Add import (non-aliased group): `"github.com/comvex-jp/digima-backend-agent/domain/services/businesstype"`.  
  
- [ ] **Step 6: Build (expect test failures, not build failures, after this)**  
  
```bash  
docker exec agent go build -buildvcs=false -o ./tmp/main .```  
Expected: build OK. (Service test files still reference the removed methods — fixed in Task 5.)  
  
- [ ] **Step 7: Commit**  
  
```bash  
git add domain/services/sms_reply/ domain/services/contact_nurturing/ app/providers/app.gogit commit -m "refactor: inject businesstype.Selector, drop duplicated helpers [DGM2-30013]"```  
  
---  
  
## Task 5: Fix the service test suites  
  
**Files:**  
- Modify: `domain/services/sms_reply/business_type_test.go` (delete moved tests)  
- Modify: `domain/services/contact_nurturing/business_type_test.go` (delete file)  
- Modify: `domain/services/contact_nurturing/service_test.go` (delete moved tests at ~920-976; inject selector where needed)  
- Modify: `domain/services/sms_reply/service_test.go` (inject selector where the resolution path runs)  
  
- [ ] **Step 1: Trim sms_reply business_type_test.go**  
  
Keep only the 5 `resolveBusinessType` tests (lines 16-44). Delete every test that calls `service.getBusinessTypeInfo`, `service.filterByTargetContactGroup`, or `selectBusinessTypeSetting` (lines 46-297). Remove now-unused imports (`context`, `errors`, `digimaAdapter`, `agentsettingmodel`, `businesstypesetting`, `mock`) — keep `testing` and `require`. Run build to confirm the exact import set.  
  
- [ ] **Step 2: Delete contact_nurturing business_type_test.go**  
  
```bash  
git rm domain/services/contact_nurturing/business_type_test.go```  
  
All its cases are now covered by `domain/services/businesstype/selector_test.go`.  
  
- [ ] **Step 3: Remove moved tests from contact_nurturing/service_test.go**  
  
Delete the business-type test functions that call `service.getBusinessTypeInfo` / `service.filterByTargetContactGroup` (the block ending at line 976, e.g. `TestGetBusinessTypeInfo_ReturnsEmptyWhenAllGatedOut` and the `filterByTargetContactGroup` test ending at 955). Search the file for `getBusinessTypeInfo` and `filterByTargetContactGroup` and remove each enclosing `func Test...`.  
  
- [ ] **Step 4: Inject the selector where the pipeline resolution path runs**  
  
Search both `domain/services/sms_reply/service_test.go` and `domain/services/contact_nurturing/service_test.go` for tests that drive the full pipeline (i.e. construct `Service{...}` with a `DigimaClient` mock and reach `tracedGetBusinessTypeInfo`). For each such `Service{...}` literal that sets `DigimaClient: <mock>`, add:  
  
```go  
       BusinessTypeSelector: businesstype.NewSelector(<mock>),  
```  
  
where `<mock>` is the same `digimaAdapter` mock variable. Add the `businesstype` import to each test file that needs it. Identify them empirically:  
  
```bash  
docker exec agent go test -p 1 --count=1 ./domain/services/sms_reply/ ./domain/services/contact_nurturing/```  
Any test that now resolves an empty business type where it previously expected a value, or fails on an unmet `GetContactGroupMemberships` expectation, needs the selector injected. (Tests whose mock had `GetContactGroupMemberships` expectations for the business-type path are the tell.)  
  
- [ ] **Step 5: Run both suites until green**  
  
Run: `docker exec agent go test -p 1 --count=1 ./domain/services/sms_reply/ ./domain/services/contact_nurturing/`  
Expected: PASS.  
  
- [ ] **Step 6: Format + commit**  
  
```bash  
docker exec agent gci write --skip-generated --custom-order -s standard -s alias -s default \  domain/services/sms_reply/business_type_test.go domain/services/sms_reply/service_test.go \  domain/services/contact_nurturing/service_test.gogit add domain/services/sms_reply/ domain/services/contact_nurturing/git commit -m "test: move business-type tests to shared package, inject selector [DGM2-30013]"```  
  
---  
  
## Task 6: Full verification  
  
- [ ] **Step 1: Build**  
  
Run: `docker exec agent go build -buildvcs=false -o ./tmp/main .`  
Expected: OK.  
  
- [ ] **Step 2: Full test suite**  
  
Run: `docker exec agent go test -p 1 --count=1 ./...`  
Expected: PASS.  
  
- [ ] **Step 3: Confirm both TODOs are gone**  
  
Run: `git grep -n "extract shared business-type\|Lift this and marshalUsageMetadata"`  
Expected: no matches.  
  
- [ ] **Step 4: Confirm no orphaned references**  
  
Run: `git grep -n "getBusinessTypeInfo\|marshalUsageMetadata\|aggregatedUsageMetadata"`  
Expected: no matches (all replaced by `Resolve` / `usagemetadata.Marshal`).  
  
- [ ] **Step 5: Final commit (if formatting changed anything)**  
  
```bash  
git add -Agit commit -m "chore: final formatting for shared business-type refactor [DGM2-30013]" || echo "nothing to commit"```  
  
---  
  
## Self-Review notes  
  
- **Spec coverage:** businesstype package (Task 3) + wiring (Task 4) ✓; usagemetadata package (Task 1) + wiring (Task 2) ✓; both TODOs removed (Tasks 2, 4; verified Task 6) ✓; tests consolidated + injected (Task 5) ✓; unified log paths (Task 3 `selector.go`) ✓; unified nil-guard = contact_nurturing semantics (Task 1 `Marshal`) ✓; sms_reply-only `resolveBusinessType` kept (Task 4 Step 3) ✓.  
- **Type consistency:** `NewSelector(client) Selector`, `Selector.Resolve(ctx, accountId, contactId, agentSetting) (string, string)`, `usagemetadata.Marshal(map[string]agentAdapter.TokenUsage) *string` — used identically everywhere.  
- **Line numbers** are from the pre-refactor snapshot; re-confirm with `grep` before each deletion since edits shift them within a file.