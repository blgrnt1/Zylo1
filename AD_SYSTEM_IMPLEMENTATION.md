# Zylo Advertisement System Implementation 📺

## 🎯 **Ad System Overview**

**Goal**: Generate $200K-500K annually through strategic advertisement placement while maintaining user experience quality.

**Key Strategy**: Native, non-intrusive ads that provide value to construction professionals.

---

## 🏗️ **Ad System Architecture**

### **Core Components**
```kotlin
// 1. Ad Manager - Central ad coordination
@Singleton
class AdManager @Inject constructor(
    private val adRepository: AdRepository,
    private val userRepository: UserRepository,
    private val analyticsService: AnalyticsService
) {
    
    suspend fun getAdsForPlacement(
        placement: AdPlacement,
        userId: String,
        limit: Int = 3
    ): List<Advertisement> {
        
        val userProfile = userRepository.getUserProfile(userId)
        val targetingCriteria = createTargetingCriteria(userProfile, placement)
        
        return adRepository.getTargetedAds(
            placement = placement,
            targeting = targetingCriteria,
            limit = limit
        ).also { ads ->
            // Track ad impressions
            ads.forEach { ad ->
                analyticsService.trackAdImpression(ad.id, placement, userId)
            }
        }
    }
    
    private fun createTargetingCriteria(
        userProfile: UserProfile,
        placement: AdPlacement
    ): AdTargeting {
        return AdTargeting(
            location = userProfile.location,
            userType = userProfile.role,
            specializations = userProfile.specializations,
            projectBudgetRange = userProfile.typicalBudgetRange,
            activityLevel = userProfile.activityScore
        )
    }
}

// 2. Ad Repository - Data management
interface AdRepository {
    suspend fun getTargetedAds(
        placement: AdPlacement,
        targeting: AdTargeting,
        limit: Int
    ): List<Advertisement>
    
    suspend fun recordAdClick(adId: String, userId: String)
    suspend fun recordAdImpression(adId: String, userId: String)
    suspend fun getAdPerformanceMetrics(adId: String): AdMetrics
}
```

### **Ad Data Models**
```kotlin
@Entity(tableName = "advertisements")
data class Advertisement(
    @PrimaryKey val id: String,
    val advertiserId: String,
    val title: String,
    val description: String,
    val imageUrl: String?,
    val clickUrl: String,
    val adType: AdType,
    val targeting: AdTargeting,
    val budget: AdBudget,
    val performance: AdPerformance,
    val status: AdStatus,
    val createdAt: Long,
    val expiresAt: Long
)

data class AdTargeting(
    val locations: List<String>,
    val userTypes: List<UserRole>,
    val specializations: List<String>,
    val budgetRange: IntRange?,
    val demographics: Demographics?
)

data class AdBudget(
    val totalBudget: Double,
    val dailyBudget: Double,
    val maxCostPerClick: Double,
    val maxCostPerImpression: Double,
    val spentToday: Double = 0.0,
    val totalSpent: Double = 0.0
)

enum class AdType {
    SPONSORED_CONTRACTOR,     // Contractors paying for visibility
    MATERIAL_SUPPLIER,        // Hardware stores, suppliers
    TOOL_RENTAL,             // Equipment rental companies
    INSURANCE_PROVIDER,      // Contractor insurance
    TRAINING_COURSE,         // Skills/certification courses
    SOFTWARE_TOOL,           // Construction management software
    BANNER_DISPLAY,          // General banner advertisements
    NATIVE_CONTENT           // Native content recommendations
}
```

---

## 📱 **Ad Placement Implementation**

### **1. Search Results Sponsored Listings**
```kotlin
@Composable
fun ContractorSearchResults(
    contractors: List<Contractor>,
    sponsoredAds: List<Advertisement>,
    onContractorClick: (String) -> Unit,
    onAdClick: (Advertisement) -> Unit
) {
    LazyColumn {
        // Sponsored section at top
        if (sponsoredAds.isNotEmpty()) {
            item {
                SponsoredSection(
                    ads = sponsoredAds,
                    onAdClick = onAdClick
                )
            }
            item { Divider(modifier = Modifier.padding(vertical = 8.dp)) }
        }
        
        // Regular contractor results
        items(contractors) { contractor ->
            ContractorCard(
                contractor = contractor,
                onClick = { onContractorClick(contractor.id) }
            )
        }
        
        // Interstitial ads every 5 results
        contractors.chunked(5).forEachIndexed { chunkIndex, chunk ->
            if (chunkIndex > 0 && sponsoredAds.isNotEmpty()) {
                item {
                    NativeAdCard(
                        ad = sponsoredAds[chunkIndex % sponsoredAds.size],
                        onClick = onAdClick
                    )
                }
            }
        }
    }
}

@Composable
fun SponsoredSection(
    ads: List<Advertisement>,
    onAdClick: (Advertisement) -> Unit
) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .padding(horizontal = 16.dp),
        colors = CardDefaults.cardColors(
            containerColor = MaterialTheme.colorScheme.primaryContainer.copy(alpha = 0.1f)
        )
    ) {
        Column(modifier = Modifier.padding(12.dp)) {
            Row(
                verticalAlignment = Alignment.CenterVertically,
                modifier = Modifier.fillMaxWidth()
            ) {
                Icon(
                    imageVector = Icons.Default.Star,
                    contentDescription = null,
                    tint = MaterialTheme.colorScheme.primary,
                    modifier = Modifier.size(16.dp)
                )
                Spacer(modifier = Modifier.width(4.dp))
                Text(
                    text = "Sponsored",
                    style = MaterialTheme.typography.labelSmall,
                    color = MaterialTheme.colorScheme.primary
                )
            }
            
            Spacer(modifier = Modifier.height(8.dp))
            
            LazyRow {
                items(ads) { ad ->
                    SponsoredContractorCard(
                        ad = ad,
                        onClick = { onAdClick(ad) }
                    )
                }
            }
        }
    }
}
```

### **2. Project Detail Page Ads**
```kotlin
@Composable
fun ProjectDetailScreen(
    project: Project,
    relatedAds: List<Advertisement>,
    viewModel: ProjectDetailViewModel = hiltViewModel()
) {
    LazyColumn {
        item { ProjectHeaderSection(project) }
        item { ProjectDescriptionSection(project) }
        item { ProjectBudgetSection(project) }
        
        // Relevant services ad section
        if (relatedAds.isNotEmpty()) {
            item {
                RelatedServicesAdSection(
                    ads = relatedAds,
                    onAdClick = { ad -> viewModel.onAdClicked(ad) }
                )
            }
        }
        
        item { ProjectTimelineSection(project) }
        item { ProjectLocationSection(project) }
    }
}

@Composable
fun RelatedServicesAdSection(
    ads: List<Advertisement>,
    onAdClick: (Advertisement) -> Unit
) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .padding(16.dp)
    ) {
        Column(modifier = Modifier.padding(16.dp)) {
            Text(
                text = "Recommended Services",
                style = MaterialTheme.typography.titleMedium,
                fontWeight = FontWeight.Bold
            )
            
            Spacer(modifier = Modifier.height(8.dp))
            
            LazyRow {
                items(ads) { ad ->
                    ServiceAdCard(
                        ad = ad,
                        onClick = { onAdClick(ad) },
                        modifier = Modifier.width(200.dp)
                    )
                }
            }
        }
    }
}
```

### **3. In-Chat Contextual Ads**
```kotlin
@Composable
fun ChatScreen(
    messages: List<Message>,
    contextualAds: List<Advertisement>,
    onSendMessage: (String) -> Unit,
    onAdClick: (Advertisement) -> Unit
) {
    LazyColumn {
        items(messages.chunked(10)) { messageChunk ->
            // Display message chunk
            messageChunk.forEach { message ->
                MessageBubble(message = message)
            }
            
            // Insert contextual ad every 10 messages
            if (contextualAds.isNotEmpty()) {
                ContextualAdBubble(
                    ad = contextualAds.random(),
                    onAdClick = onAdClick
                )
            }
        }
    }
}

@Composable
fun ContextualAdBubble(
    ad: Advertisement,
    onAdClick: (Advertisement) -> Unit
) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .padding(horizontal = 16.dp, vertical = 4.dp)
            .clickable { onAdClick(ad) },
        colors = CardDefaults.cardColors(
            containerColor = MaterialTheme.colorScheme.surfaceVariant.copy(alpha = 0.5f)
        )
    ) {
        Row(
            modifier = Modifier.padding(12.dp),
            verticalAlignment = Alignment.CenterVertically
        ) {
            AsyncImage(
                model = ad.imageUrl,
                contentDescription = null,
                modifier = Modifier.size(40.dp)
            )
            
            Spacer(modifier = Modifier.width(12.dp))
            
            Column(modifier = Modifier.weight(1f)) {
                Text(
                    text = ad.title,
                    style = MaterialTheme.typography.bodyMedium,
                    fontWeight = FontWeight.Medium
                )
                Text(
                    text = ad.description,
                    style = MaterialTheme.typography.bodySmall,
                    color = MaterialTheme.colorScheme.onSurface.copy(alpha = 0.7f)
                )
            }
            
            Text(
                text = "Ad",
                style = MaterialTheme.typography.labelSmall,
                color = MaterialTheme.colorScheme.outline,
                modifier = Modifier
                    .background(
                        MaterialTheme.colorScheme.outline.copy(alpha = 0.1f),
                        RoundedCornerShape(4.dp)
                    )
                    .padding(horizontal = 6.dp, vertical = 2.dp)
            )
        }
    }
}
```

---

## 💰 **Revenue Calculation & Tracking**

### **Ad Performance Analytics**
```kotlin
@Entity(tableName = "ad_performance")
data class AdPerformance(
    @PrimaryKey val adId: String,
    val impressions: Int = 0,
    val clicks: Int = 0,
    val conversions: Int = 0,
    val revenue: Double = 0.0,
    val ctr: Double = 0.0, // Click-through rate
    val cpc: Double = 0.0, // Cost per click
    val cpm: Double = 0.0, // Cost per mille (1000 impressions)
    val lastUpdated: Long
) {
    fun calculateCTR(): Double = if (impressions > 0) (clicks.toDouble() / impressions) * 100 else 0.0
    fun calculateCPC(): Double = if (clicks > 0) revenue / clicks else 0.0
    fun calculateCPM(): Double = if (impressions > 0) (revenue / impressions) * 1000 else 0.0
}

class AdAnalyticsService @Inject constructor(
    private val adPerformanceDao: AdPerformanceDao,
    private val firebaseAnalytics: FirebaseAnalytics
) {
    
    suspend fun trackAdImpression(adId: String, placement: AdPlacement, userId: String) {
        // Update local performance data
        adPerformanceDao.incrementImpressions(adId)
        
        // Track in Firebase Analytics
        firebaseAnalytics.logEvent("ad_impression") {
            param("ad_id", adId)
            param("placement", placement.name)
            param("user_id", userId)
        }
    }
    
    suspend fun trackAdClick(adId: String, placement: AdPlacement, userId: String): Double {
        val performance = adPerformanceDao.getAdPerformance(adId)
        val clickRevenue = calculateClickRevenue(adId, placement)
        
        // Update performance metrics
        adPerformanceDao.incrementClicks(adId, clickRevenue)
        
        // Track revenue
        firebaseAnalytics.logEvent("ad_click") {
            param("ad_id", adId)
            param("placement", placement.name)
            param("revenue", clickRevenue)
            param("user_id", userId)
        }
        
        return clickRevenue
    }
    
    private suspend fun calculateClickRevenue(adId: String, placement: AdPlacement): Double {
        val ad = getAdvertisement(adId)
        val baseRate = placement.pricePerClick
        
        // Dynamic pricing based on targeting and competition
        val targetingMultiplier = calculateTargetingMultiplier(ad.targeting)
        val competitionMultiplier = calculateCompetitionMultiplier(placement, ad.adType)
        
        return baseRate * targetingMultiplier * competitionMultiplier
    }
}
```

### **Revenue Optimization Engine**
```kotlin
class AdRevenueOptimizer @Inject constructor(
    private val adRepository: AdRepository,
    private val performanceRepository: AdPerformanceRepository
) {
    
    suspend fun optimizeAdPlacement(): AdOptimizationResult {
        val allAds = adRepository.getActiveAds()
        val performanceData = performanceRepository.getRecentPerformance(7) // Last 7 days
        
        val optimizedPlacements = mutableListOf<AdPlacementOptimization>()
        
        AdPlacement.values().forEach { placement ->
            val placementAds = allAds.filter { it.placement == placement }
            val placementPerformance = performanceData.filter { it.placement == placement }
            
            val optimization = optimizePlacement(placement, placementAds, placementPerformance)
            optimizedPlacements.add(optimization)
        }
        
        return AdOptimizationResult(
            totalRevenueIncrease = optimizedPlacements.sumOf { it.projectedRevenueIncrease },
            optimizations = optimizedPlacements
        )
    }
    
    private fun optimizePlacement(
        placement: AdPlacement,
        ads: List<Advertisement>,
        performance: List<AdPerformanceData>
    ): AdPlacementOptimization {
        
        // A/B test different ad configurations
        val currentRevenue = performance.sumOf { it.revenue }
        val averageCTR = performance.map { it.ctr }.average()
        
        // Recommend optimizations
        val recommendations = mutableListOf<String>()
        
        if (averageCTR < 2.0) {
            recommendations.add("Improve ad creative and targeting")
        }
        
        if (ads.size < 5) {
            recommendations.add("Increase ad inventory for this placement")
        }
        
        return AdPlacementOptimization(
            placement = placement,
            currentRevenue = currentRevenue,
            projectedRevenueIncrease = currentRevenue * 0.15, // 15% improvement target
            recommendations = recommendations
        )
    }
}
```

---

## 📊 **Ad Performance Dashboard**

### **Admin Dashboard for Ad Management**
```kotlin
@Composable
fun AdDashboardScreen(
    viewModel: AdDashboardViewModel = hiltViewModel()
) {
    val adMetrics by viewModel.adMetrics.collectAsState()
    val revenueData by viewModel.revenueData.collectAsState()
    
    LazyColumn {
        item {
            AdRevenueOverviewCard(
                totalRevenue = revenueData.totalRevenue,
                dailyRevenue = revenueData.dailyRevenue,
                growth = revenueData.growthPercentage
            )
        }
        
        item {
            AdPerformanceMetricsCard(
                impressions = adMetrics.totalImpressions,
                clicks = adMetrics.totalClicks,
                ctr = adMetrics.averageCTR,
                cpc = adMetrics.averageCPC
            )
        }
        
        item {
            TopPerformingAdsSection(
                topAds = adMetrics.topPerformingAds
            )
        }
        
        item {
            AdPlacementPerformanceChart(
                placementData = adMetrics.placementPerformance
            )
        }
    }
}

@Composable
fun AdRevenueOverviewCard(
    totalRevenue: Double,
    dailyRevenue: Double, 
    growth: Double
) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .padding(16.dp)
    ) {
        Column(modifier = Modifier.padding(16.dp)) {
            Text(
                text = "Ad Revenue Overview",
                style = MaterialTheme.typography.titleLarge,
                fontWeight = FontWeight.Bold
            )
            
            Spacer(modifier = Modifier.height(16.dp))
            
            Row(
                modifier = Modifier.fillMaxWidth(),
                horizontalArrangement = Arrangement.SpaceBetween
            ) {
                RevenueMetric(
                    label = "Total Revenue",
                    value = "$${"%.2f".format(totalRevenue)}",
                    icon = Icons.Default.AttachMoney
                )
                
                RevenueMetric(
                    label = "Daily Revenue", 
                    value = "$${"%.2f".format(dailyRevenue)}",
                    icon = Icons.Default.TrendingUp
                )
                
                RevenueMetric(
                    label = "Growth",
                    value = "${"%.1f".format(growth)}%",
                    icon = Icons.Default.ShowChart,
                    isPositive = growth > 0
                )
            }
        }
    }
}
```

---

## 🎯 **Advertiser Onboarding**

### **Self-Service Ad Creation**
```kotlin
@Composable
fun CreateAdCampaignScreen(
    onCampaignCreated: (AdCampaign) -> Unit,
    viewModel: CreateAdViewModel = hiltViewModel()
) {
    val uiState by viewModel.uiState.collectAsState()
    
    LazyColumn {
        item {
            AdBasicInfoSection(
                title = uiState.title,
                description = uiState.description,
                onTitleChange = viewModel::updateTitle,
                onDescriptionChange = viewModel::updateDescription
            )
        }
        
        item {
            AdTargetingSection(
                targeting = uiState.targeting,
                onTargetingChange = viewModel::updateTargeting
            )
        }
        
        item {
            AdBudgetSection(
                budget = uiState.budget,
                onBudgetChange = viewModel::updateBudget
            )
        }
        
        item {
            AdPreviewSection(
                adPreview = uiState.toAdPreview()
            )
        }
        
        item {
            CreateCampaignButton(
                enabled = uiState.isValid,
                onClick = { viewModel.createCampaign() }
            )
        }
    }
}
```

This comprehensive ad system will generate significant revenue while providing value to construction professionals through relevant, targeted advertisements. The key is maintaining user experience quality while maximizing ad performance and revenue!
