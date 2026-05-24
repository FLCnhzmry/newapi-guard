<template>
  <article class="policy-group-card" :class="{ root }" :style="{ '--policy-indent': depth }">
    <div class="policy-group-head">
      <div>
        <div class="capsule-label">{{ root ? "根规则" : "规则组" }}</div>
        <div class="policy-group-title">{{ root ? "Discord 准入规则" : `${logicText(group.logic)} 规则组` }}</div>
      </div>
      <div class="policy-group-head-actions">
        <label class="policy-inline-control policy-logic-control">
          <span>{{ root ? "根关系" : "组内关系" }}</span>
          <select v-model="group.logic" class="app-select policy-select">
            <option value="and">AND</option>
            <option value="or">OR</option>
          </select>
        </label>
        <button v-if="!root" class="mini-action danger" type="button" @click="$emit('remove-group')">删除组</button>
      </div>
    </div>

    <p class="policy-group-note">
      {{ root ? "没有任何条件或子组时，表示全部放行。" : "当前组会作为一个独立条目参与父组判断。" }}
    </p>

    <div class="policy-condition-list">
      <div v-for="(condition, index) in group.conditions" :key="condition.id" class="policy-condition-row">
        <div class="policy-condition-head">
          <div class="policy-condition-type">{{ conditionKind(condition) === "roles" ? "身份组规则" : "服务器规则" }}</div>
          <button class="mini-action danger" type="button" @click="removeCondition(index)">删除</button>
        </div>

        <label class="policy-inline-control policy-kind-control">
          <span>条件类型</span>
          <select :value="conditionKind(condition)" class="app-select policy-select" @change="changeConditionKind(condition, $event.target.value)">
            <option value="guild_id">服务器 ID</option>
            <option value="roles">角色 ID</option>
          </select>
        </label>

        <label class="policy-inline-control policy-value-control">
          <span>条件值</span>
          <input
            v-model="condition.value"
            class="app-input policy-input"
            :placeholder="placeholderFor(condition)"
            type="text"
            autocomplete="off"
          />
        </label>

        <div class="policy-condition-tag">{{ conditionLabel(condition) }}</div>
      </div>

      <div v-if="!group.conditions.length" class="policy-empty-line">当前组里还没有规则。</div>
    </div>

    <div class="policy-group-children">
      <PolicyGroupEditor
        v-for="(child, index) in group.groups"
        :key="child.id"
        :group="child"
        :depth="depth + 1"
        :root="false"
        @remove-group="removeGroup(index)"
      />
    </div>

    <div class="policy-toolbar">
      <button class="mini-action" type="button" @click="addCondition('guild_id')">新增服务器规则</button>
      <button class="mini-action" type="button" @click="addCondition('roles')">新增身份组规则</button>
      <button class="mini-action" type="button" @click="addGroup('and')">新增 AND 组</button>
      <button class="mini-action" type="button" @click="addGroup('or')">新增 OR 组</button>
    </div>
  </article>
</template>

<script setup>
defineOptions({ name: "PolicyGroupEditor" });

const props = defineProps({
  group: {
    type: Object,
    required: true
  },
  depth: {
    type: Number,
    default: 0
  },
  root: {
    type: Boolean,
    default: false
  }
});

defineEmits(["remove-group"]);

function createId() {
  if (typeof crypto !== "undefined" && typeof crypto.randomUUID === "function") {
    return crypto.randomUUID();
  }
  return `policy-${Math.random().toString(16).slice(2)}-${Date.now()}`;
}

function createCondition(kind = "guild_id") {
  if (kind === "roles") {
    return {
      id: createId(),
      field: "roles",
      op: "contains",
      value: ""
    };
  }
  return {
    id: createId(),
    field: "guild_id",
    op: "eq",
    value: ""
  };
}

function createGroup(logic = "and") {
  return {
    id: createId(),
    logic: logic === "or" ? "or" : "and",
    conditions: [],
    groups: []
  };
}

function logicText(logic) {
  return logic === "or" ? "OR" : "AND";
}

function conditionKind(condition) {
  return condition.field === "roles" ? "roles" : "guild_id";
}

function conditionLabel(condition) {
  return conditionKind(condition) === "roles" ? "包含角色" : "服务器匹配";
}

function placeholderFor(condition) {
  return conditionKind(condition) === "roles" ? "输入角色 ID" : "输入服务器 ID";
}

function changeConditionKind(condition, kind) {
  if (kind === "roles") {
    condition.field = "roles";
    condition.op = "contains";
    return;
  }
  condition.field = "guild_id";
  condition.op = "eq";
}

function addCondition(kind) {
  props.group.conditions.push(createCondition(kind));
}

function removeCondition(index) {
  props.group.conditions.splice(index, 1);
}

function addGroup(logic) {
  props.group.groups.push(createGroup(logic));
}

function removeGroup(index) {
  props.group.groups.splice(index, 1);
}
</script>
