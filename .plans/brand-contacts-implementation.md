# 品牌联系方式功能实施计划

## 📋 任务概述

**目标：** 为品牌添加联系方式功能，与店铺、产品保持一致

**优先级：** P1（高优先级）

**预计工时：** 1-2 天

**负责人：** Claude Team

---

## 🎯 需求范围

### 需要实现的功能

1. ✅ **后端 API**
   - 数据库添加 contacts 字段
   - 实体类添加 contacts 字段
   - API 支持联系方式的增删改查

2. ✅ **后台管理**
   - 品牌编辑器添加联系方式表格
   - 使用 BatchEditTable 组件
   - 与店铺、产品联系方式样式保持一致

3. ✅ **C 端展示**
   - 品牌详情页展示联系方式
   - 显示姓名、电话、邮箱

---

## 🛠️ 技术实现

### 一、后端实现（casalyin-java）

#### 1. 数据库迁移

**文件：** `casalyin-admin/src/main/resources/db/migration/V28__add_brand_contacts.sql`

**内容：**
```sql
-- 添加品牌联系方式字段
ALTER TABLE t_brand ADD COLUMN contacts JSON COMMENT '联系方式列表：[{name, phone, email}]';
ALTER TABLE t_brand_draft ADD COLUMN contacts JSON COMMENT '联系方式列表：[{name, phone, email}]';
```

#### 2. 实体类修改

**文件：** `casalyin-admin/src/main/java/com/casalyin/admin/module/busniess/brand/domain/entity/BrandEntity.java`

**添加字段：**
```java
/**
 * 联系方式列表：[{name, phone, email}]
 */
private String contacts;
```

**文件：** `casalyin-admin/src/main/java/com/casalyin/admin/module/busniess/brand/domain/entity/BrandDraftEntity.java`

**添加字段：**
```java
/**
 * 联系方式列表：[{name, phone, email}]
 */
private String contacts;
```

#### 3. DTO 修改

**文件：** `casalyin-admin/src/main/java/com/casalyin/admin/module/busniess/brand/domain/dto/ContactDTO.java`

**内容：**
```java
package com.casalyin.admin.module.busniess.brand.domain.dto;

import io.swagger.v3.oas.annotations.media.Schema;
import lombok.Data;

import java.io.Serializable;

@Data
@Schema(description = "联系方式 DTO")
public class ContactDTO implements Serializable {
    private static final long serialVersionUID = 1L;

    @Schema(description = "姓名")
    private String name;

    @Schema(description = "电话")
    private String phone;

    @Schema(description = "邮箱")
    private String email;
}
```

#### 4. Form 修改

**文件：** `casalyin-admin/src/main/java/com/casalyin/admin/module/busniess/brand/domain/form/BrandAddForm.java`

**添加字段：**
```java
@Schema(description = "联系方式列表：name/email/phone 至少填一个")
private List<ContactDTO> contacts;
```

**文件：** `casalyin-admin/src/main/java/com/casalyin/admin/module/busniess/brand/domain/form/BrandUpdateForm.java`

**添加字段：**
```java
@Schema(description = "联系方式列表：name/email/phone 至少填一个")
private List<ContactDTO> contacts;
```

**文件：** `casalyin-admin/src/main/java/com/casalyin/admin/module/busniess/brand/domain/form/BrandDraftUpdateForm.java`

**添加字段：**
```java
@Schema(description = "联系方式列表：name/email/phone 至少填一个")
private List<ContactDTO> contacts;
```

#### 5. VO 修改

**文件：** `casalyin-admin/src/main/java/com/casalyin/admin/module/busniess/brand/domain/vo/BrandVO.java`

**添加字段：**
```java
@Schema(description = "联系方式列表：[{name?, email?, phone?}]")
private List<ContactDTO> contacts;
```

**文件：** `casalyin-admin/src/main/java/com/casalyin/admin/module/busniess/brand/domain/vo/BrandDraftVO.java`

**添加字段：**
```java
@Schema(description = "联系方式列表：[{name?, email?, phone?}]")
private List<ContactDTO> contacts;
```

#### 6. Service 修改

**文件：** `casalyin-admin/src/main/java/com/casalyin/admin/module/busniess/brand/service/BrandService.java`

**修改点：**
- 在创建/更新品牌时，处理 contacts 字段（JSON 序列化）
- 在查询品牌时，解析 contacts 字段（JSON 反序列化）

**文件：** `casalyin-admin/src/main/java/com/casalyin/admin/module/busniess/brand/service/BrandDraftService.java`

**修改点：**
- 在创建/更新草稿时，处理 contacts 字段
- 在对比草稿差异时，添加 contacts 字段的对比

#### 7. Mapper 修改

**文件：** `casalyin-admin/src/main/resources/mapper/BrandMapper.xml`

**修改点：**
- 在查询语句中添加 contacts 字段
- 确保所有查询都包含 contacts 字段

---

### 二、后台前端实现（casalyin-server）

#### 1. 品牌编辑器修改

**文件：** `casalyin-server/src/pages/BrandEditor.tsx`

**修改内容：**

1. **导入必要的组件和类型**
```tsx
import BatchEditTable from '../components/common/BatchEditTable'
import type { ColumnsType } from 'antd/es/table'
```

2. **定义类型**
```tsx
interface ContactInfo {
  name: string
  phone: string
  email?: string
}
```

3. **添加状态管理**
```tsx
const [contactsList, setContactsList] = useState<ContactInfo[]>([])
const watchedContacts = Form.useWatch('contacts', form)

useEffect(() => {
  if (Array.isArray(watchedContacts)) {
    setContactsList(watchedContacts)
  }
}, [watchedContacts])
```

4. **定义列配置函数**
```tsx
const buildContactsColumns = (editable: boolean): ColumnsType<ContactInfo> => [
  {
    title: t('brandForm.contactName'),
    dataIndex: 'name',
    width: 200,
    render: (text, record, index) => (
      editable ? (
        <Input
          value={text}
          onChange={(e) => {
            const newList = [...contactsList]
            newList[index] = { ...newList[index], name: e.target.value }
            setContactsList(newList)
            form.setFieldValue('contacts', newList)
          }}
          placeholder={t('brandForm.contactNamePlaceholder')}
        />
      ) : text || '-'
    ),
  },
  {
    title: t('brandForm.contactPhone'),
    dataIndex: 'phone',
    width: 200,
    render: (text, record, index) => (
      editable ? (
        <Input
          value={text}
          onChange={(e) => {
            const newList = [...contactsList]
            newList[index] = { ...newList[index], phone: e.target.value }
            setContactsList(newList)
            form.setFieldValue('contacts', newList)
          }}
          placeholder={t('brandForm.contactPhonePlaceholder')}
        />
      ) : text || '-'
    ),
  },
  {
    title: t('brandForm.contactEmail'),
    dataIndex: 'email',
    width: 200,
    render: (text, record, index) => (
      editable ? (
        <Input
          value={text}
          onChange={(e) => {
            const newList = [...contactsList]
            newList[index] = { ...newList[index], email: e.target.value }
            setContactsList(newList)
            form.setFieldValue('contacts', newList)
          }}
          placeholder={t('brandForm.contactEmailPlaceholder')}
        />
      ) : text || '-'
    ),
  },
  {
    title: t('common.actions'),
    width: 100,
    render: (_, record, index) => (
      editable ? (
        <Button
          type="link"
          danger
          onClick={() => {
            const newList = contactsList.filter((_, i) => i !== index)
            setContactsList(newList)
            form.setFieldValue('contacts', newList)
          }}
        >
          {t('common.delete')}
        </Button>
      ) : null
    ),
  },
]
```

5. **添加联系方式表格**
```tsx
<Form.Item label={t('brandForm.contacts')} style={{ marginBottom: 16 }}>
  <Form.Item name="contacts" noStyle>
    <div style={{ display: 'none' }} />
  </Form.Item>
  <BatchEditTable<ContactInfo>
    columns={buildContactsColumns(!readonly)}
    dataSource={contactsList}
    rowKey={(record, index) => `contact-${index}`}
    onAddRow={!readonly ? () => {
      const newList = [...contactsList, { name: '', phone: '', email: '' }]
      setContactsList(newList)
      form.setFieldValue('contacts', newList)
    } : undefined}
    addRowLabel={t('brandForm.addContact')}
    scroll={{ x: 700 }}
  />
</Form.Item>
```

#### 2. 翻译文件修改

**文件：** `casalyin-server/src/locales/zh-CN.json`

**添加翻译：**
```json
"brandForm": {
  "contacts": "联系方式",
  "contactName": "姓名",
  "contactPhone": "电话",
  "contactEmail": "邮箱",
  "contactNamePlaceholder": "请输入姓名",
  "contactPhonePlaceholder": "请输入电话",
  "contactEmailPlaceholder": "请输入邮箱",
  "addContact": "添加联系方式"
}
```

**文件：** `casalyin-server/src/locales/en.json`

**添加翻译：**
```json
"brandForm": {
  "contacts": "Contacts",
  "contactName": "Name",
  "contactPhone": "Phone",
  "contactEmail": "Email",
  "contactNamePlaceholder": "Enter name",
  "contactPhonePlaceholder": "Enter phone",
  "contactEmailPlaceholder": "Enter email",
  "addContact": "Add Contact"
}
```

**文件：** `casalyin-server/src/locales/es-ES.json`

**添加翻译：**
```json
"brandForm": {
  "contacts": "Contactos",
  "contactName": "Nombre",
  "contactPhone": "Teléfono",
  "contactEmail": "Correo electrónico",
  "contactNamePlaceholder": "Ingrese nombre",
  "contactPhonePlaceholder": "Ingrese teléfono",
  "contactEmailPlaceholder": "Ingrese correo electrónico",
  "addContact": "Agregar Contacto"
}
```

---

### 三、C 端前端实现（casalyin-Headless）

#### 1. 品牌详情页修改

**文件：** `casalyin-Headless/src/app/[locale]/brands/[slug]/page.tsx`

**修改内容：**

1. **添加联系方式展示区域**
```tsx
{/* 联系方式 */}
{brand.contacts && brand.contacts.length > 0 && (
  <section className="mb-8">
    <h2 className="text-2xl font-bold mb-4">联系方式</h2>
    <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
      {brand.contacts.map((contact, index) => (
        <div key={index} className="border rounded-lg p-4">
          {contact.name && (
            <div className="mb-2">
              <span className="font-semibold">姓名：</span>
              <span>{contact.name}</span>
            </div>
          )}
          {contact.phone && (
            <div className="mb-2">
              <span className="font-semibold">电话：</span>
              <a href={`tel:${contact.phone}`} className="text-blue-600 hover:underline">
                {contact.phone}
              </a>
            </div>
          )}
          {contact.email && (
            <div>
              <span className="font-semibold">邮箱：</span>
              <a href={`mailto:${contact.email}`} className="text-blue-600 hover:underline">
                {contact.email}
              </a>
            </div>
          )}
        </div>
      ))}
    </div>
  </section>
)}
```

2. **类型定义**
```tsx
interface ContactInfo {
  name?: string
  phone?: string
  email?: string
}

interface Brand {
  // ... 其他字段
  contacts?: ContactInfo[]
}
```

---

## 📂 文件修改清单

### 后端（casalyin-java）

**数据库：**
- [ ] `casalyin-admin/src/main/resources/db/migration/V28__add_brand_contacts.sql`

**实体类：**
- [ ] `casalyin-admin/src/main/java/com/casalyin/admin/module/busniess/brand/domain/entity/BrandEntity.java`
- [ ] `casalyin-admin/src/main/java/com/casalyin/admin/module/busniess/brand/domain/entity/BrandDraftEntity.java`

**DTO：**
- [ ] `casalyin-admin/src/main/java/com/casalyin/admin/module/busniess/brand/domain/dto/ContactDTO.java`（新建）

**Form：**
- [ ] `casalyin-admin/src/main/java/com/casalyin/admin/module/busniess/brand/domain/form/BrandAddForm.java`
- [ ] `casalyin-admin/src/main/java/com/casalyin/admin/module/busniess/brand/domain/form/BrandUpdateForm.java`
- [ ] `casalyin-admin/src/main/java/com/casalyin/admin/module/busniess/brand/domain/form/BrandDraftUpdateForm.java`

**VO：**
- [ ] `casalyin-admin/src/main/java/com/casalyin/admin/module/busniess/brand/domain/vo/BrandVO.java`
- [ ] `casalyin-admin/src/main/java/com/casalyin/admin/module/busniess/brand/domain/vo/BrandDraftVO.java`

**Service：**
- [ ] `casalyin-admin/src/main/java/com/casalyin/admin/module/busniess/brand/service/BrandService.java`
- [ ] `casalyin-admin/src/main/java/com/casalyin/admin/module/busniess/brand/service/BrandDraftService.java`

**Mapper：**
- [ ] `casalyin-admin/src/main/resources/mapper/BrandMapper.xml`

### 后台前端（casalyin-server）

- [ ] `casalyin-server/src/pages/BrandEditor.tsx`
- [ ] `casalyin-server/src/locales/zh-CN.json`
- [ ] `casalyin-server/src/locales/en.json`
- [ ] `casalyin-server/src/locales/es-ES.json`

### C 端前端（casalyin-Headless）

- [ ] `casalyin-Headless/src/app/[locale]/brands/[slug]/page.tsx`
- [ ] `casalyin-Headless/src/types/brand.ts`（如果有）

---

## ✅ 验收标准

### 后端验收

1. **数据库**
   - [ ] t_brand 表添加 contacts 字段
   - [ ] t_brand_draft 表添加 contacts 字段
   - [ ] 字段类型为 JSON

2. **API**
   - [ ] 创建品牌时可以保存联系方式
   - [ ] 更新品牌时可以修改联系方式
   - [ ] 查询品牌时返回联系方式
   - [ ] 草稿功能支持联系方式

### 后台前端验收

1. **品牌编辑器**
   - [ ] 可以添加新的联系方式
   - [ ] 可以编辑姓名、电话、邮箱
   - [ ] 可以删除联系方式
   - [ ] 表格占据全宽度
   - [ ] 添加按钮在表格下方
   - [ ] 只读模式显示正确
   - [ ] 与店铺、产品联系方式样式一致

### C 端前端验收

1. **品牌详情页**
   - [ ] 显示联系方式列表
   - [ ] 显示姓名、电话、邮箱
   - [ ] 电话可点击拨打
   - [ ] 邮箱可点击发送邮件
   - [ ] 响应式布局正常

---

## 🚀 实施计划

### 第一阶段：后端实现（0.5-1 天）

**Agent 1：后端开发**
- 创建数据库迁移文件
- 修改实体类、DTO、Form、VO
- 修改 Service 和 Mapper
- 自测 API 功能

### 第二阶段：后台前端实现（0.5 天）

**Agent 2：后台前端开发**
- 修改品牌编辑器
- 添加联系方式表格
- 添加翻译文件
- 自测功能

### 第三阶段：C 端前端实现（0.5 天）

**Agent 3：C 端前端开发**
- 修改品牌详情页
- 添加联系方式展示
- 自测功能

### 第四阶段：测试和 Review（0.5 天）

**Agent 4：测试和 Review**
- 集成测试
- Code Review
- 修复问题

---

## 📝 注意事项

1. **数据格式统一**
   - 联系方式使用 JSON 格式存储：`[{name, phone, email}]`
   - 与店铺、产品保持一致

2. **样式统一**
   - 使用 `BatchEditTable` 组件
   - 与店铺、产品联系方式样式完全一致

3. **国际化**
   - 所有文案使用 i18n
   - 支持中文、英文、西班牙语

4. **兼容性**
   - 确保旧数据兼容（contacts 字段为空时不报错）
   - 确保 API 向后兼容

---

**创建时间：** 2026-05-03

**最后更新：** 2026-05-03

**状态：** 待开始

**优先级：** P1（高优先级）
