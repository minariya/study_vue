<template>
    <div>
        <div
            class="group_container open_group_container"
            @click="clickGroup(groupData, openGroupData)"
            :style="{ width : `${width}px`, height: `${height}px`}"
        >
            <img :src="groupData.thumbnailUrl" alt="썸네일" @error="imageError" v-if="!isErrorImg" />
            <div v-else class="error-tumbnail">
                썸네일 오류입니다 바꿔주세요 !!
            </div>
        </div>
        <div class="outside_area">
            <div class="main_text">
                <span>{{groupData.groupName}}</span>
                <div class="like-area">
                    <img
                        class="img_area"
                        v-if="showLikes"
                        :src="isLike ? likeIcon : notIcon"
                        @click="clickLikeGroup(openGroupData)"
                        alt="하트"
                    />
                    <div class="user-number" v-if="likeUserCount > 0">{{ likeUserCount }}</div>
                    <div class="bar" v-if="likeUserCount > 0">|</div>
                    <div class="user-number">{{ musicLength }}곡</div>
                </div>
            </div>
            <div class="sub_text">{{groupData.description}}</div>
        </div>
    </div>
</template>

<script>
import { computed, ref, watch } from "@vue/composition-api";
import {
    getGroupListByKey,
    addLikeGroup,
    deleteLikeGroup
} from "@/services/group";
import { getMusicListByGroup } from '@/services/music';

const getGroupData = (userInfo, openGroupData) => {
    const groupData = ref({});
    const getImage = ref(null);
    const musicLength = ref(0);

    const data = {
        dealiName: openGroupData.dealiName,
        key: openGroupData.targetKey,
        groupKey: openGroupData.targetKey,
    };
    getGroupListByKey(data).on("value", snapshot => {
        if (!snapshot.val()) return;
        groupData.value = snapshot.val();
    });

    getMusicListByGroup(data).once('value', snapshot => {
        if (!snapshot.val()) return musicLength.value = 0;
        musicLength.value = Object.keys(snapshot.val()).length;
    });
    return {
        groupData,
        getImage,
        musicLength,
    };
};

const clickEvent = (userInfo, router, isLike, likeUserCount) => {
    const clickGroup = (groupData, openGroupData) => {
        router.push({
            path: "/playPage",
            query: {
                groupHost: openGroupData.dealiName,
                groupName: groupData.groupName,
                groupKey: groupData.targetKey || groupData.myKey
            }
        });
    };

    const clickLikeGroup = (openGroupData) => {
        if (!userInfo.value.dealiName) {
            alert("로그인 후 하트 눌러주세요 !");
            return;
        }
        const data = {
            dealiName: userInfo.value.dealiName,
            targetName: openGroupData.dealiName,
            targetKey: openGroupData.targetKey,
            isShowGroup: true
        };
        if (isLike.value === false) {
            addLikeGroup(data);
            isLike.value = true;
            likeUserCount.value += 1;
        } else {
            deleteLikeGroup(data);
            isLike.value = false;
            likeUserCount.value -= 1;
        }
    };
    return {
        clickGroup,
        clickLikeGroup
    };
};


const iconList = () => {
    const likeIcon = require("@/assets/icons/Icon_heart_18x18(x2).png");
    const notIcon = require("@/assets/icons/Icon_heart_outline18x18(x2).png");

    return {
        likeIcon,
        notIcon
    };
};

export default {
    name: "OpenGroup",
    props: {
        openGroupData: {
            type: [String, Object]
        },
        width: {
            type: String,
            default: "240"
        },
        height: {
            type: String,
            default: "180"
        },
        showLikes: {
            type: Boolean,
            default: true
        }
    },
    setup(props, { root }) {
        const userInfo = computed(
            () => root.$store.getters["login/getUserStatus"]
        );
        const isLike = ref(false);
        const likeUser = computed(() => props.openGroupData.likes);
        const likeUserCount = ref(0);
        const isErrorImg = ref(false);
        if (props.openGroupData.likes) likeUserCount.value = props.openGroupData.likes.count;

        const imageError = () => isErrorImg.value = true;

        watch(
            () => userInfo.value.dealiName,
            () => {
                if (!likeUser.value) return;
                if (
                    Object.keys(likeUser.value).includes(
                        userInfo.value.dealiName
                    )
                ) {
                    isLike.value = true;
                } else {
                    isLike.value = false;
                }
            }
        );

        return {
            ...getGroupData(userInfo, props.openGroupData),
            isLike,
            ...iconList(),
            ...clickEvent(userInfo, root.$router, isLike, likeUserCount),
            likeUserCount,
            imageError,
            isErrorImg,
        };
    }
};
</script>

<style lang="scss" scoped>
.group_container {
    box-sizing: border-box;
    color: $White;
    position: relative;
    cursor: pointer;

    .error-tumbnail{
        display: flex;
        align-items: center;
        justify-content: center;
        color: $White;
        height: 100%;
        width: 100%;
    }
}
.outside_area {
    color: $Black;

    .main_text {
        display: flex;
        justify-content: space-between;
        align-items: center;
        font-size: 15px;
        font-weight: bold;
        margin: 16px 0 8px;
        color: $White;

        .like-area {
            display: flex;
            align-items: center;
            justify-content: flex-end;
            color: $Gray400;
            width: 70px;

            .img_area {
                width: 15px;
                cursor: pointer;
                margin-right: 5px;
            }
            .user-number{
                display: flex;
                align-items: center;
                justify-content: center;
                font-size: 12px;
                font-weight: normal;
            }
            .bar{
                margin: 0 5px;
                font-size: 12px;
            }
        }
    }
    .sub_text {
        font-size: 12px;
        color: $Gray600;
    }
}
</style>