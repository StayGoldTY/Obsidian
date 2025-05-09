根据我rules里面配置的# Business Logic信息，加上我后端代码里面定义的model对象字段，加上我 @常用语句精确版.sql 里面的信息，以及我配置的mssql 的mcp工具，生成对应的统计sql 语句，记住有变更完成的数据，以变更为准。

-- 年度施工图审查绿色建筑面积统计（优化版 - 处理项目变更）
WITH ProjectData AS (
    SELECT 
        a.ID,
        a.preID,  -- 前一个项目ID（如果是变更项目）
        a.ProjCode,
        a.ProjName,
        a.UPDATETIME,
        YEAR(a.UPDATETIME) AS 统计年度
    FROM 
        T_HN_Main a
        INNER JOIN WF_Instance ins ON a.ID = ins.MainId
            AND ins.Status = '10'  -- 已完成的工作流
            AND ins.FatherId IS NULL
            AND ins.WF_Code IN ('WF_0013', 'WF_0015', 'WF_0016', 'WF_0017')  -- 施工图审查相关工作流
    WHERE
        a.AccountCode NOT IN ('jianshe', 'sheji')  -- 排除系统账号
        AND a.UPDATETIME BETWEEN '2020-01-01' AND '2023-12-31'  -- 统计2020-2023年
        AND a.ProjStatus = '10'  -- 有效项目
),
-- 标记每个项目（包括变更项目）的排序，只保留最新版本
RankedProjects AS (
    SELECT 
        *,
        ROW_NUMBER() OVER (
            PARTITION BY 
                CASE WHEN preID IS NULL THEN ID ELSE preID END  -- 按原始项目ID分组
            ORDER BY 
                UPDATETIME DESC  -- 按更新时间降序排序
        ) AS ProjectRank
    FROM 
        ProjectData
),
-- 获取有效项目（最新版本且未被变更的）
ValidProjects AS (
    SELECT 
        ID,
        ProjCode,
        ProjName,
        统计年度
    FROM 
        RankedProjects
    WHERE 
        ProjectRank = 1  -- 只保留最新版本
        AND ID NOT IN (  -- 排除被变更的项目
            SELECT preID 
            FROM ProjectData 
            WHERE preID IS NOT NULL
        )
),
-- 获取绿色建筑数据
GreenBuildingData AS (
    SELECT 
        vp.统计年度,
        vp.ID,
        vp.ProjName,
        sb.ID AS 单体ID,
        sb.BS_BuildingName AS 单体名称,
        sb.BS_BuildingArea AS 单体总建筑面积,
        
        -- 获取绿色建筑信息
        CASE JSON_VALUE(CAST(sb.BS_BuildingJson AS NVARCHAR(MAX)), '$.IsHaveGreenBuilding')
            WHEN '1' THEN 1
            ELSE 0
        END AS 是否绿色建筑,
        
        -- 处理绿色建筑面积
        CASE 
            WHEN JSON_VALUE(CAST(sb.BS_BuildingJson AS NVARCHAR(MAX)), '$.IsHaveGreenBuilding') = '1' 
            THEN TRY_CAST(JSON_VALUE(CAST(sb.BS_BuildingJson AS NVARCHAR(MAX)), '$.GreenBuildingArea') AS NUMERIC(18,2))
            ELSE 0
        END AS 单体绿色建筑面积,
        
        -- 获取绿色建筑星级
        CASE JSON_VALUE(CAST(sb.BS_BuildingJson AS NVARCHAR(MAX)), '$.GreenBuildingGrade')
            WHEN '0' THEN 0   -- 基本级
            WHEN '1' THEN 1   -- 一星级
            WHEN '2' THEN 2   -- 二星级
            WHEN '3' THEN 3   -- 三星级
            ELSE NULL        -- 非绿色建筑或未指定星级
        END AS 绿色建筑星级
    FROM 
        ValidProjects vp
        LEFT JOIN T_HN_Main_SingleBidSection sb ON vp.ID = sb.MainID
    WHERE
        JSON_VALUE(CAST(sb.BS_BuildingJson AS NVARCHAR(MAX)), '$.IsHaveGreenBuilding') = '1'  -- 是绿色建筑
        AND JSON_VALUE(CAST(sb.BS_BuildingJson AS NVARCHAR(MAX)), '$.GreenBuildingArea') IS NOT NULL  -- 绿色建筑面积不为空
),
YearlyStats AS (
    SELECT
        统计年度,
        -- 计算总面积和各星级面积
        SUM(单体绿色建筑面积) AS 当年施工图审查绿色建筑总面积,
        SUM(CASE WHEN 绿色建筑星级 = 1 THEN 单体绿色建筑面积 ELSE 0 END) AS 当年施工图审查中一星级绿色建筑面积,
        SUM(CASE WHEN 绿色建筑星级 = 2 THEN 单体绿色建筑面积 ELSE 0 END) AS 当年施工图审查中二星级绿色建筑面积,
        SUM(CASE WHEN 绿色建筑星级 = 3 THEN 单体绿色建筑面积 ELSE 0 END) AS 当年施工图审查中三星级绿色建筑面积
    FROM 
        GreenBuildingData
    GROUP BY 
        统计年度
)
-- 生成最终报表，包括所有年份（2020-2023）
SELECT 
    CAST(Y.年份 AS VARCHAR(10)) + '年' AS 年度,
    ISNULL(CAST(ROUND(S.当年施工图审查绿色建筑总面积/10000, 2) AS DECIMAL(18,2)), 0) AS 当年施工图审查绿色建筑总面积_万平方米,
    ISNULL(CAST(ROUND(S.当年施工图审查中一星级绿色建筑面积/10000, 2) AS DECIMAL(18,2)), 0) AS 当年施工图审查中一星级绿色建筑面积_万平方米,
    ISNULL(CAST(ROUND(S.当年施工图审查中二星级绿色建筑面积/10000, 2) AS DECIMAL(18,2)), 0) AS 当年施工图审查中二星级绿色建筑面积_万平方米,
    ISNULL(CAST(ROUND(S.当年施工图审查中三星级绿色建筑面积/10000, 2) AS DECIMAL(18,2)), 0) AS 当年施工图审查中三星级绿色建筑面积_万平方米
FROM 
    (VALUES (2020), (2021), (2022), (2023)) AS Y(年份)
LEFT JOIN 
    YearlyStats S ON Y.年份 = S.统计年度
ORDER BY 
    Y.年份;