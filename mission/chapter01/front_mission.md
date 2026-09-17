type Role = "LEADER" | "MEMBER";

type StudyMember = {
id: number;
name: string;
role: Role;
githubId?: string;
};

const members: StudyMember[] = [
{ id: 1, name: "유리", role: "LEADER", githubId: "Kohseoyoung" },
{ id: 2, name: "진진", role: "MEMBER" },
{ id: 3, name: "제로", role: "MEMBER" },
];

function findMember(id: number): StudyMember | undefined {
return members.find((member) => member.id === id);
}

function createMemberCard(id: number): string {
const member = findMember(id);

if (member === undefined) {
return id + "번 회원을 찾을 수 없습니다.";
}

const roleName = member.role === "LEADER" ? "스터디장" : "스터디원";
const github = member.githubId ?? "미등록";

return `${member.name} 님 (${roleName}) / GitHub: ${github}`;
}

console.log(createMemberCard(1));
console.log(createMemberCard(2));
console.log(createMemberCard(3));
console.log(createMemberCard(999));